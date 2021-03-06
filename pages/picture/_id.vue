<template>
  <div class="page">
    <template v-if="status === 200">
      <div :style="{ height: `${100 * proportion}vw` }" class="img-box">
        <img v-lazy="$img.secdraLazy(picture, 500, true)" />
      </div>
      <template v-if="picture.user.id !== user.id">
        <div class="flex-text user-info-box">
          <nuxt-link v-ripple :to="`/user/${picture.user.id}`" class="head-box">
            <img
              v-lazy="$img.headLazy(picture.user.head, 'small50')"
              :title="picture.user.name"
            />
          </nuxt-link>
          <nuxt-link
            :to="`/user/${picture.user.id}`"
            class="nickname primary-hover"
          >
            {{ picture.user.name }}
          </nuxt-link>
          <Btn
            @click="follow(picture.user.id)"
            block
            color="primary"
            class="following-btn"
          >
            {{
              picture.user.focus === $enum.FollowState.CONCERNED.key
                ? `已关注`
                : `关注`
            }}
          </Btn>
        </div>
        <div class="line"></div>
      </template>
      <div class="info-box">
        <h3 class="name">
          <strong>{{ picture.name }}</strong>
        </h3>
        <p class="introduction">{{ picture.introduction }}</p>
        <div class="flex-text tool-row">
          <div class="flex-text">
            <Btn flat icon small title="浏览">
              <i class="icon ali-icon-attention"></i>
            </Btn>
            <span>{{ picture.viewAmount }}</span>
          </div>
          <div class="flex-text">
            <Btn
              :color="
                picture.focus === $enum.CollectState.CONCERNED.key
                  ? `primary`
                  : `default`
              "
              @click="collection(picture)"
              flat
              icon
              small
              title="收藏"
            >
              <i
                :class="{
                  'ali-icon-likefill':
                    picture.focus === $enum.CollectState.CONCERNED.key,
                  'ali-icon-like':
                    picture.focus !== $enum.CollectState.CONCERNED.key
                }"
                class="icon"
              ></i>
            </Btn>
            <span>{{ picture.likeAmount }}</span>
          </div>
        </div>
        <div class="date">创建于：{{ picture.createDate | date }}</div>
      </div>
      <div class="line"></div>
      <div class="tag-list">
        <Tag
          v-for="(item, index) in picture.tagList"
          :key="index"
          :content="item"
          @click="$router.push(`/picture/search/${encodeURIComponent(item)}`)"
          class="tag"
          color="primary"
          clickable
        ></Tag>
      </div>
      <CornerButtons>
        <Btn
          v-if="picture.user.id === user.id"
          @click="editShow = true"
          icon
          big
          shadow
          color="white"
        >
          <i class="icon ali-icon-edit"></i>
        </Btn>
        <Btn v-else @click="collection(picture)" icon big shadow color="white">
          <i
            :class="{
              'ali-icon-likefill':
                picture.focus === $enum.CollectState.CONCERNED.key,
              'ali-icon-like':
                picture.focus !== $enum.CollectState.CONCERNED.key,
              'primary-text': picture.focus === $enum.CollectState.CONCERNED.key
            }"
            class="icon"
          ></i> </Btn
      ></CornerButtons>
      <Model v-model="editShow" v-loading="editLoading">
        <div class="edit-model">
          <header class="edit-header">
            <nav class="flex-text">
              <Btn @click="editShow = false" icon flat>
                <i class="icon ali-icon-back"></i>
              </Btn>
              <div class="title center">
                编辑
              </div>
              <Btn @click="editShow = false" icon flat>
                <i class="icon ali-icon-edit"></i>
              </Btn>
            </nav>
          </header>
        </div>
      </Model>
    </template>
    <template v-else-if="status === 403">
      <div class="empty">
        <img src="../../assets/image/svg/default-picture.svg" />
      </div>
    </template>
    <template v-else-if="status === 404">
      <div class="empty">
        <img src="../../assets/image/svg/default-picture.svg" />
      </div>
    </template>
  </div>
</template>

<script>
import Model from "../../components/global/Model"
import Tag from "../../components/global/Tag"
import CornerButtons from "../../components/pages/shared/CornerButtons"
import { CommentForm } from "../../assets/script/model"
import { mapActions, mapState, mapMutations } from "vuex"
export default {
  components: {
    Model,
    Tag,
    CornerButtons
  },
  data() {
    return {
      editShow: false,
      editLoading: false,
      inputTag: ""
    }
  },
  computed: {
    ...mapState("user", ["user"]),
    signedIn() {
      return this.user && this.user.id
    },
    proportion() {
      return this.picture.height / this.picture.width
    }
  },
  async asyncData({ store, route, $axios }) {
    store.commit("menu/MChangeName", "detail")
    const id = route.params.id
    const res = await $axios.get(`/picture/get`, {
      params: { id }
    })
    const result = res.data
    let pictureForm
    let inputTag
    let commentForm
    if (result.status === 200) {
      pictureForm = Object.assign({}, result.data)
      inputTag = pictureForm.tagList.join(" ")
      commentForm = new CommentForm(result.data.user.id, result.data.id)
    }
    return {
      status: result.status,
      picture: result.data,
      pictureForm,
      inputTag,
      commentForm
    }
  },
  head() {
    let title = "想你所想 - Secdra"
    if (this.status === 200) {
      title = this.picture.name + " - Secdra"
    } else if (this.status === 403) {
      title = "无权查看该图片"
      this.MChangeTitle(title)
    } else if (this.status === 404) {
      title = "图片不存在"
      this.MChangeTitle(title)
    }
    return { title }
  },
  mounted() {
    // 写入足迹
    this.status === 200 &&
      this.signedIn &&
      this.ASaveFootprint({ pictureId: this.picture.id })
  },
  methods: {
    ...mapMutations("menu", ["MChangeTitle"]),
    ...mapActions("picture", ["ACollection", "AUpdate"]),
    ...mapActions("footprint", { ASaveFootprint: "ASave" }),
    ...mapActions("user", ["AFollow"]),
    async collection(picture) {
      const result = await this.ACollection({
        pictureId: picture.id
      })
      if (result.status !== 200) {
        this.$tooltip({ message: result.message })
        return
      }
      picture.focus = result.data
    },
    async follow(id) {
      const result = await this.AFollow({
        followingId: id
      })
      if (result.status !== 200) {
        this.$tooltip({ message: result.message })
        return
      }
      this.picture.user.focus = result.data
    },
    async save() {
      this.editLoading = true
      const tagList = this.inputTag.split(" ").filter((item) => item !== "")
      this.pictureForm.tagList = [...new Set(tagList)]
      const result = await this.AUpdate(this.pictureForm)
      this.editLoading = false
      if (result.status !== 200) {
        this.$tooltip({ message: result.message })
        return
      }
      this.$tooltip({ message: `修改成功` })
      this.editShow = false
      this.picture = result.data
      this.reset()
    },
    reset() {
      this.pictureForm = Object.assign({}, this.picture)
    }
  }
}
</script>

<style type="text/less" lang="less" scoped>
@import "../../assets/style/color";
@import "../../assets/style/config";
@import "../../assets/style/mixin";
.page {
  min-height: 100vh;
  margin-top: 0;
  .img-box {
    width: 100vw;
    display: flex;
    align-items: center;
    img {
      object-fit: cover;
      width: 100%;
      height: 100%;
    }
  }
  @padding-size: 4vw;
  .user-info-box {
    @info-box-height: 14vw;
    @img-size: @info-box-height - @padding-size;
    @follow-btn-size: 20vw;
    padding: @padding-size;
    .head-box {
      display: block;
      position: relative;
      border-radius: 50%;
      img {
        border-radius: 50%;
        width: @img-size;
        height: @img-size;
      }
    }
    .nickname {
      padding: 0 6vw;
      flex: 1;
      .ellipsis();
    }
    .following-btn {
      padding: 0;
      width: @follow-btn-size;
    }
  }
  .info-box {
    padding: @padding-size;
    .name {
      .ellipsis();
    }
    @margin-top: 2vw;
    .introduction {
      margin-top: @margin-top;
      font-size: @smallest-font-size;
      color: @font-color-dark-fade;
    }
    .tool-row {
      margin-top: @margin-top;
      > div {
        flex: 1;
      }
    }
    .date {
      margin-top: @margin-top;
      font-size: @smallest-font-size;
      color: @font-color-dark-fade;
    }
  }
  .tag-list {
    padding: @padding-size;
    .tag {
      margin-right: 1.5vw;
      margin-bottom: 1.5vw;
    }
  }
  .edit-model {
    @padding: 3vw;
    width: 100%;
    height: @herder-height;
    font-size: @big-font-size;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    box-shadow: 0 0 0.2vw 0 rgba(0, 0, 0, 0.2);
    z-index: 10;
    user-select: none;
    nav {
      width: 100%;
      margin: 0 auto;
      font-size: 0;
      height: @herder-height;
      line-height: @herder-height;
      padding: 0 @padding;
      background-color: white;
      .title {
        flex: 1;
        font-size: @big-font-size;
        color: @font-color-dark;
        .ellipsis();
      }
    }
  }
  .empty {
    height: 100vh;
    width: @window-min-width;
    img {
      height: 100%;
      width: 100%;
      object-fit: cover;
    }
  }
}
</style>
