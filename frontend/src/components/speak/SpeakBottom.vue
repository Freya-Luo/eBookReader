<template>
  <div class="bottom-wrapper" :class="{'hide-play': !showPlay}">
    <div class="bottom-playing-wrapper" @click.stop.prevent="onPlayingCardClick" v-if="showPlay">
      <div class="bottom-playing-left">
        <div class="icon-play-wrapper" @click.stop.prevent="togglePlay">
          <span class="icon-play_go" v-if="!isPlaying"></span>
          <span class="icon-play_pause" v-else></span>
        </div>
      </div>
      <div class="bottom-playing-right">
        <div class="bottom-playing-chapter-text">
          <span class="chapter-label">{{chapter ? chapter.label : ''}}</span>
          <span
            class="bottom-playing-page-text"
            v-if="currentSectionIndex && currentSectionTotal"
          >( {{currentSectionIndex}} / {{currentSectionTotal}} )</span>
        </div>
        <div
          class="bottom-playing-page-text"
        >{{playInfo ? playInfo.currentMinute : '00'}}:{{playInfo ? playInfo.currentSecond : '00'}} / {{playInfo ? playInfo.totalMinute : '00'}}:{{playInfo ? playInfo.totalSecond : '00'}}</div>
      </div>
    </div>
    <div class="bottom-btn-wrapper" @click.stop.prevent="addOrRemove()">
      <span class="bottom-btn-text">
        <span class="icon-check" v-if="inBookShelf"></span>
        {{inBookShelf ? $t('detail.isAddedToShelf') : $t('detail.addOrRemoveShelf')}}
      </span>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import { storeShelfMixin } from '../../utils/mixin'
import {
  removeBookFromShelf,
  addBookToShelf,
  findBookFromList
} from '../../utils/store'
import { getBookShelf, saveBookShelf } from '../../utils/localStorage'
import { flatList } from '../../api/store'

export default {
  mixins: [storeShelfMixin],
  data() {
    return {
      bookItem: null,
      fileName: ''
    }
  },
  props: {
    chapter: Object,
    currentSectionIndex: Number,
    currentSectionTotal: Number,
    showPlay: Boolean,
    isPlaying: Boolean,
    playInfo: Object
  },
  computed: {
    inBookShelf() {
      if (this.bookItem && this.shelfList) {
        const flatShelf = (function flatten(arr) {
          return [].concat(
            ...arr.map((v) => (v.itemList ? [v, ...flatten(v.itemList)] : v))
          )
        })(this.shelfList).filter((item) => item.type === 1)
        const book = flatShelf.filter(
          (item) => item.fileName === this.bookItem.fileName
        )
        return book && book.length > 0
      } else {
        return false
      }
    }
  },
  methods: {
    togglePlay() {
      // call togglePlay() in the StoreSpeaking component
      this.$parent.togglePlay()
    },
    onPlayingCardClick() {
      this.$emit('onPlayingCardClick')
    },
    async findBookFromList() {
      let book
       await flatList().then((response) => {
        if (response.status === 200) {
          const bookList = response.data.data.filter(
            (item) => item.fileName === this.fileName
          )
          if (bookList && bookList.length > 0) {
            book = bookList[0]
          }
        }
      })
      this.bookItem = book
    },
    addOrRemove() {
      if (this.inBookShelf) {
        this.setShelfList(removeBookFromShelf(this.bookItem)).then(() => {
          saveBookShelf(this.shelfList)
        })
      } else {
        addBookToShelf(this.bookItem)
        this.setShelfList(getBookShelf())
      }
    }
  },
  mounted() {
    this.fileName = this.$route.query.fileName
    this.findBookFromList()
  }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
@import '../../assets/styles/global';

.bottom-wrapper {
  position: absolute;
  bottom: 0;
  left: 0;
  z-index: 110;
  width: 100%;
  height: px2rem(116);
  background: #faf3dd;
  box-shadow: 0 px2rem(-2) px2rem(2) 0 rgba(0, 0, 0, 0.1);
  &.hide-play {
    height: px2rem(52);
  }
  .bottom-playing-wrapper {
    display: flex;
    width: 100%;
    height: px2rem(64);
    .bottom-playing-left {
      flex: 0 0 px2rem(70);
      width: px2rem(70);
      @include center;
      .icon-play-wrapper {
        flex: 0 0 px2rem(100);
        @include center;
        &:active {
          opacity: 0.5;
        }
        .icon-play_go {
          font-size: px2rem(40);
          color: $color-blue;
        }
        .icon-play_pause {
          font-size: px2rem(40);
          color: $color-blue;
        }
      }
    }
    .bottom-playing-right {
      flex: 1;
      padding-right: px2rem(15);
      box-sizing: border-box;
      @include columnLeft;
      .bottom-playing-chapter-text {
        max-width: px2rem(305);
        color: #333;
        font-size: px2rem(16);
        line-height: px2rem(19);
        @include ellipsis;
      }
      .bottom-playing-page-text {
        color: #666;
        font-size: px2rem(10);
        margin-top: px2rem(5);
      }
    }
  }
  .bottom-btn-wrapper {
    width: 100%;
    height: px2rem(52);
    border-top: px2rem(1) solid #eee;
    box-sizing: border-box;
    @include center;
    .bottom-btn-text {
      font-size: px2rem(14);
      color: $color-blue;
      font-weight: bold;
    }
  }
}
</style>
