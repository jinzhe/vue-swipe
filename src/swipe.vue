<template>
  <div class="swipe" :style="swipeStyle" v-if="count>0" @touchmove.prevent>
    <div class="swipe-items" :style="swipeItemsStyle">
      <div
        class="swipe-item"
        v-for="(item,index) in items"
        :key="index"
        :style="{'background-image':'url('+item+')','width':width+'px'}"
        @click="$emit('click')"
      ></div>
    </div>
    <span class="indicator">
      <span
        v-for="(_,$index) in count"
        :key="$index"
        :class="{ active: index==$index }"
        @click="to($index)"
      ></span>
    </span>
  </div>
</template>

<script>
export default {
  props: {
    autoplay: { type: Boolean, default: false },
    width: { type: Number, default: window.innerWidth },
    height: { type: Number, default: window.innerHeight },
    //滑动所需要的时间
    items: {
      type: Array,
      default: []
    }
  },
  data() {
    return {
      index: 0,
      timer: null,
      run: false
    };
  },
  computed: {
    // swipe 容器
    swipeStyle: function() {
      return {
        width: this.width + "px",
        height: this.height + "px"
      };
    },
    // slide 父容器
    swipeItemsStyle: function() {
      let itemWidth = this.width || window.innerWidth;
      let offset = -itemWidth * this.index;
      let width = itemWidth * this.items.length;
      return {
        width: width + "px",
        height: this.height + "px",
        transform: "translate3d(" + offset + "px,0,0)"
      };
    },
    // slide 总数
    count: function() {
      return this.items.length;
    }
  },
  mounted() {
    // 初始化
    this.$nextTick(_ => {
      this.setAutoplay();
      let target = this.$el.querySelector(".swipe-items");
      let cacheOffset = 0;

      this.swipe(
        target,
        direction => {
          // 每次松手回调处理
          if (direction == "right") {
            this.index--;
            if (this.index <= 0) {
              this.index = 0;
            }
          } else if (direction == "left") {
            this.index++;
            if (this.index >= this.count - 1) {
              this.index = this.count - 1;
            }
          }
          this.to(this.index);
          this.setAutoplay();
        },
        {
          // 记录按下的时候缓存偏移值
          start: () => {
            this.run = true;
            target.style.transition = "none";
            cacheOffset = target.style.transform.match(/(-?\d*)px/)[1] >> 0;
          },
          // 手指移动的时候
          move: offset => {
            let friction = 0;
            if (this.index == 0 && offset > 0) {
              friction = Math.pow(Math.sqrt(offset * 0.4), 2);
            } else if (this.index == this.count - 1 && offset < 0) {
              friction = -Math.pow(Math.sqrt(Math.abs(offset) * 0.4), 2);
            } else {
              friction = offset;
            }
            target.style.transform =
              "translate3d(" + (cacheOffset + friction) + "px,0,0)";
          },
          // 松手的时候
          end: () => {
            this.run = false;
            target.style.transition =
              "transform 250ms cubic-bezier(0.25, 0.46, 0.45, 0.94)";
            // 恢复
            let itemWidth = this.width;
            let offset = -itemWidth * this.index;
            target.style.transform = "translate3d(" + offset + "px,0,0)";
          }
        }
      );
    });
  },
  methods: {
    // 转到指定索引
    to(index) {
      this.index = index;
      this.setAutoplay();
    },
    // 自动播放
    setAutoplay() {
      if (!this.autoplay) {
        return;
      }
      clearInterval(this.timer);
      this.timer = setInterval(() => {
        if (this.run) {
          return false;
        }
        this.index++;
        if (this.index >= this.count) {
          this.index = 0;
        }
      }, 5000);
    },
    // 手势操作
    swipe(node, callback, options = {}) {
      let st;
      let sTouch;
      let eTouch;
      let defaultOptions = {
        direction: "x", //x or y
        speed: 200,
        offset: 10,
        prevent: true
      };
      // 继承默认参数
      Object.entries(defaultOptions).forEach(([key, value]) => {
        if (!options[key]) {
          options[key] = value;
        }
      });
      if (window.ontouchstart === undefined) {
        node.onmousedown = function(e) {
          if (options.prevent) {
            e.preventDefault();
          }
          st = e.timeStamp;
          sTouch = e;
          options.start && options.start();
          if (typeof options.move === "function") {
            document.onmousemove = function(e) {
              if (!sTouch) {
                return;
              }
              eTouch = e;
              if (options.direction === "x") {
                options.move(e.pageX - sTouch.pageX);
              } else {
                options.move(e.pageY - sTouch.pageY);
              }
            };
          }

          document.onmouseup = function(e) {
            this.onmousemove = null;
            this.onmouseup = null;
            eTouch = e;
            let time = e.timeStamp - st;
            let offset;
            let direction;

            if (options.direction === "x") {
              offset = eTouch.pageX - sTouch.pageX;
              direction = offset > 0 ? "right" : "left";
            } else {
              offset = eTouch.pageY - sTouch.pageY;
              direction = offset > 0 ? "down" : "up";
            }
            if (
              Math.abs(offset) >= options.offset ||
              (Math.abs(offset) / time) * 1000 >= options.speed
            ) {
              callback && callback(direction);
            }
            options.end && options.end();
          };
        };
      } else {
        node.addEventListener(
          "touchstart",
          e => {
            if (options.prevent) {
              e.preventDefault();
            }
            st = e.timeStamp;
            sTouch = eTouch = e.targetTouches[0];
            options.start && options.start();
          },
          false
        );

        if (typeof options.move === "function") {
          node.addEventListener(
            "touchmove",
            e => {
              eTouch = e.targetTouches[0];
              if (options.direction === "x") {
                options.move(eTouch.pageX - sTouch.pageX);
              } else {
                options.move(eTouch.pageY - sTouch.pageY);
              }
            },
            false
          );
        }

        node.addEventListener(
          "touchend",
          e => {
            eTouch = e.changedTouches[0];
            let time = e.timeStamp - st;
            let offset;
            let direction;

            if (options.direction === "x") {
              offset = eTouch.pageX - sTouch.pageX;
              direction = offset > 0 ? "right" : "left";
            } else {
              offset = eTouch.pageY - sTouch.pageY;
              direction = offset > 0 ? "down" : "up";
            }
            if (
              Math.abs(offset) >= options.offset ||
              (Math.abs(offset) / time) * 1000 >= options.speed
            ) {
              callback && callback(direction);
            }
            options.end && options.end();
          },
          false
        );
      }
    }
  }
};
</script>

<style lang="less" scoped>
.swipe {
  position: relative;
  overflow: hidden;
  margin: auto;
  background-color: rgba(0, 0, 0, 0.01);
  & > div {
    height: 100%;
    transition: transform 250ms cubic-bezier(0.25, 0.46, 0.45, 0.94);
    & > div {
      float: left;
      width: 100vw;
      height: 100%;
      background-repeat: no-repeat;
      background-size: cover;
      background-position: center;
    }
  }
  .indicator {
    display: block;
    position: absolute;
    bottom: 0px;
    width: 100%;
    height: 25px;
    text-align: center;
    background: 0 0;
    span {
      display: inline-block;
      width: 8px;
      height: 8px;
      margin: 1px 7px;
      cursor: pointer;
      border-radius: 100%;
      background: rgba(255, 255, 255, 0.5);
      &.active {
        background: #08c;
      }
    }
  }
}
</style>