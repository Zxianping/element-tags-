<template>
  <div class="tags-nav">
    <div class="close-con">
      <el-dropdown transfer @click="handleTagsOption" style="margin-top:7px;">
        <el-button size="small" type="text">
         
        </el-button>
        <el-dropdown-menu slot="list">
          <el-dropdown-item name="close-all">关闭所有</el-dropdown-item>
          <el-dropdown-item name="close-others">关闭其他</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>

<ul v-show="visible" :style="{left: contextMenuLeft + 'px', top: contextMenuTop + 'px'}" class="contextmenu">
      <li v-for="(item, key) of menuList" @click="handleTagsOption(key)" :key="key">{{item}}</li>
    </ul>
    <div class="btn-con left-btn">
      <el-button type="primary" @click="handleScroll(240)">
        
      </el-button>
    </div>
    <div class="btn-con right-btn">
      <el-button type="primary" @click="handleScroll(-240)">
       
      </el-button>
    </div>
      <div class="scroll-outer" ref="scrollOuter" @DOMMouseScroll="handlescroll" @mousewheel="handlescroll">
      <div ref="scrollBody" class="scroll-body" :style="{left: tagBodyLeft + 'px'}">
        <el-tag ref='tagsPageOpened' class="tags-view-item"   :data-route-item="tag" :type="isActive(tag)?'success':''" v-for="tag in Array.from(visitedViews)" :key="tag.path" @contextmenu.prevent.native="openMenu(tag,$event)" @click.native="handleClick(tag)">
{{generateTitle(tag.title)}}
 <span class='el-icon-close' @click.prevent.stop='closeSelectedTag(tag)'></span>
        </el-tag>
         <!-- <router-link ref='tag' class="tags-view-item" :class="isActive(tag)?'active':''" v-for="tag in Array.from(visitedViews)" :to="tag.path" :key="tag.path" @contextmenu.prevent.native="openMenu(tag,$event)">
        {{generateTitle(tag.title)}}
       
        <span class='el-icon-close' @click.prevent.stop='closeSelectedTag(tag)'></span>
      </router-link> -->
      </div>
      </div>

     <!-- <div class='tags-view-wrapper scroll-container' ref='scrollContainer' @wheel.prevent="handleScroll">
       <div class="scroll-wrapper" ref="scrollWrapper" :style="{left: lefts + 'px'}">
      <router-link ref='tag' class="tags-view-item" :class="isActive(tag)?'active':''" v-for="tag in Array.from(visitedViews)" :to="tag.path" :key="tag.path" @contextmenu.prevent.native="openMenu(tag,$event)">
        {{generateTitle(tag.title)}}
        <span class='el-icon-close' @click.prevent.stop='closeSelectedTag(tag)'></span>
      </router-link>
       </div>
    </div> -->
  
  </div>
    
</template>

<script>
const padding = 15;
import ScrollPane from "@/components/ScrollPane";
import { generateTitle } from "@/utils/i18n";
import { routeEqual } from "@/utils/utils"

export default {
  components: { ScrollPane },
  data() {
    return {
      // visible: false,
      top: 0,
      left: 0,
      lefts: 0,
      selectedTag: {},
       tagBodyLeft: 0,
      rightOffset: 40,
      outerPadding: 4,
      contextMenuLeft: 0,
      contextMenuTop: 0,
      visible: false,
      menuList: {
        others: '关闭其他',
        all: '关闭所有'
      }
    
    };
  },
  computed: {
    visitedViews() {
      return this.$store.state.tagsView.visitedViews;
    }
  },
  watch: {
    $route(to) {
      
      this.addViewTags();
      // this.moveToCurrentTag();
      this.getTagElementByRoute(to)
      
    },
    visible(value) {
      if (value) {
        window.addEventListener("click", this.closeMenu);
      } else {
        window.removeEventListener("click", this.closeMenu);
      }
    }
  },
  mounted() {
    this.addViewTags();
     setTimeout(() => {
      this.getTagElementByRoute(this.$route)
    }, 200)
  },
  methods: {
    handleClick(tag){  //跳转
// console.log(tag)
this.$router.push({name:tag.name})
    },
    handleTagsOption(){ //关闭哪个

    },
     closeMenu () {
      this.visible = false
    },
    handlescroll (e) {
      var type = e.type
      let delta = 0
      if (type === 'DOMMouseScroll' || type === 'mousewheel') {
        delta = (e.wheelDelta) ? e.wheelDelta : -(e.detail || 0) * 40
      }
      this.handleScroll(delta)
    },
    handleScroll (offset) {
      const outerWidth = this.$refs.scrollOuter.offsetWidth
      const bodyWidth = this.$refs.scrollBody.offsetWidth
      if (offset > 0) {
        this.tagBodyLeft = Math.min(0, this.tagBodyLeft + offset)
      } else {
        if (outerWidth < bodyWidth) {
          if (this.tagBodyLeft < -(bodyWidth - outerWidth)) {
            this.tagBodyLeft = this.tagBodyLeft
          } else {
            this.tagBodyLeft = Math.max(this.tagBodyLeft + offset, outerWidth - bodyWidth)
          }
        } else {
          this.tagBodyLeft = 0
        }
      }
    },
    moveToTarget(tag){
 const outerWidth = this.$refs.scrollOuter.offsetWidth
      const bodyWidth = this.$refs.scrollBody.offsetWidth
      if (bodyWidth < outerWidth) {
        this.tagBodyLeft = 0
      } else if (tag.offsetLeft < -this.tagBodyLeft) {
        // 标签在可视区域左侧
        this.tagBodyLeft = -tag.offsetLeft + this.outerPadding
      } else if (tag.offsetLeft > -this.tagBodyLeft && tag.offsetLeft + tag.offsetWidth < -this.tagBodyLeft + outerWidth) {
        // 标签在可视区域
        this.tagBodyLeft = Math.min(0, outerWidth - tag.offsetWidth - tag.offsetLeft - this.outerPadding)
      } else {
        // 标签在可视区域右侧
        this.tagBodyLeft = -(tag.offsetLeft - (outerWidth - this.outerPadding - tag.offsetWidth))
      }
    },
     getTagElementByRoute (route) {
      
      this.$nextTick(() => {
        this.refsTag = this.$refs.tagsPageOpened
        
        this.refsTag.forEach((item, index) => {
          if (routeEqual(route, item.$attrs['data-route-item'])) {
            let tag = this.refsTag[index].$el
            this.moveToTarget(tag)
          }
        })
      })
    },
    // handleScroll(e) {
    //   const eventDelta = e.wheelDelta || -e.deltaY * 3;
    //   const $container = this.$refs.scrollContainer;
    //   const $containerWidth = $container.offsetWidth;
    //   const $wrapper = this.$refs.scrollWrapper;
    //   const $wrapperWidth = $wrapper.offsetWidth;

    //   if (eventDelta > 0) {
    //     this.lefts = Math.min(0, this.lefts + eventDelta);
    //   } else {
    //     if ($containerWidth - padding < $wrapperWidth) {
    //       if (this.lefts < -($wrapperWidth - $containerWidth + padding)) {
    //         this.lefts = this.lefts;
    //       } else {
    //         this.lefts = Math.max(
    //           this.lefts + eventDelta,
    //           $containerWidth - $wrapperWidth - padding
    //         );
    //       }
    //     } else {
    //       this.lefts = 0;
    //     }
    //   }
    // },
    // moveToTarget($target) {
    //   const $container = this.$refs.scrollContainer;
    //   const $containerWidth = $container.offsetWidth;
    //   const $targetLeft = $target.offsetLeft;
    //   const $targetWidth = $target.offsetWidth;

    //   if ($targetLeft < -this.left) {
    //     // tag in the left
    //     this.left = -$targetLeft + padding;
    //   } else if (
    //     $targetLeft + padding > -this.left &&
    //     $targetLeft + $targetWidth < -this.left + $containerWidth - padding
    //   ) {
    //     // tag in the current view
    //     // eslint-disable-line
    //   } else {
    //     // tag in the right
    //     this.left = -($targetLeft - ($containerWidth - $targetWidth) + padding);
    //   }
    // },
    generateTitle, // generateTitle by vue-i18n
    generateRoute() {
      let matched = [...this.$route.matched];
      matched.splice(0, 1);
      matched = matched.filter(item => item.name);
      if (matched) {
        return matched;
      }
      return false;
    },
    isActive(route) {
      return route.path === this.$route.path || route.name === this.$route.name;
    },
    addViewTags() {
      const routes = this.generateRoute();
      if (!routes) {
        return false;
      }
      const length = routes.length;
      routes.forEach((item, index) => {
        if (index === length - 1) {
          item.showInVisitedViews = true;
        }
        this.$store.dispatch("addVisitedViews", item);
      });
    },
    
    // moveToCurrentTag() {
    //   const tags = this.$refs.tag;
    //   this.$nextTick(() => {
    //     for (const tag of tags) {
    //       if (tag.to === this.$route.path) {
    //         this.moveToTarget(tag.$el);
    //         // this.$refs.scrollPane.moveToTarget(tag.$el)
    //         break;
    //       }
    //     }
    //   });
    // },
    closeSelectedTag(view) {
      this.$store.dispatch("delVisitedViews", view).then(views => {
        if (this.isActive(view)) {
          const latestView = views.slice(-1)[0];
          if (latestView) {
            this.$router.push(latestView.path);
          } else {
            this.$router.push("/");
          }
        }
      });
    },
    closeOthersTags() {
      this.$router.push(this.selectedTag.path);
      this.$store.dispatch("delOthersViews", this.selectedTag).then(() => {
        this.moveToCurrentTag();
      });
    },
    closeAllTags() {
      this.$store.dispatch("delAllViews");
      this.$router.push("/");
    },
    openMenu(tag, e) {  //右键菜单
      // this.visible = true;
      // this.selectedTag = tag;
      // this.left = e.clientX;
      // this.top = e.clientY;
      //  if (item.name === this.$config.homeName) {
      //   return
      // }
      this.visible = true
      const offsetLeft = this.$el.getBoundingClientRect().left
      this.contextMenuLeft = e.clientX - offsetLeft + 10
      this.contextMenuTop = e.clientY - 64
    },
    // closeMenu() {
    //   this.visible = false;
    // }
  }
};
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.tags-nav {
  position: relative;
  .close-con {
    position: absolute;
    top: 0;
    z-index: 10;
  }
  .contextmenu{
    position: absolute;
    padding:5px 0;
    z-index: 1000;
    li{
      padding:5px 15px;
      cursor: pointer;
    }
  }
  .btn-con{
    padding-top:3px;
    position: absolute;
    top:0;
    z-index: 10;
  }
  .left-btn{
    left: 0;
  }
  .right-btn{
    right: 0;
  }
  .scroll-outer{
    position: absolute;
    left:28px;
    right:61px;
    top:0;
    bottom: 0;
    .scroll-body{
      height: calc(100% - 1px);
    display: inline-block;
    padding: 1px 4px 0;
    position: absolute;
    overflow: visible;
    white-space: nowrap;
    }
  }
  
}
</style>

