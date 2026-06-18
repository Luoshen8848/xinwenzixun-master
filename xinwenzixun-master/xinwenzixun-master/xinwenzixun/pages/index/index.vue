
<template>
  <view class="page">
    <!-- tab平铺导航 -->
    <view class="tab-bar">
      <view
        v-for="(tab, index) in tabs"
        :key="index"
        class="tab-item"
        :class="{ active: currentTab === index }"
        @click="changeTab(index)"
      >
        {{ tab }}
      </view>
    </view>

    <!-- 新闻列表 -->
    <view class="list">
      <view
        v-for="(item, index) in filteredList"
        :key="item.id"
        class="card"
        @click="goDetail(item.id)"
      >
        <image class="cover" :src="item.cover" mode="aspectFill"></image>
        <view class="info">
          <text class="title">{{ item.title }}</text>
          <view class="meta">
            <text class="meta-text">{{ item.source }}</text>
            <text class="meta-text">评论 {{ item.commentCount }}</text>
            <text class="meta-text">{{ item.time }}</text>
          </view>
        </view>
        <text class="close" @click.stop="removeItem(index)">×</text>
      </view>

      <!-- 空状态 -->
      <view v-if="filteredList.length === 0" class="empty">
        <text>暂无内容</text>
      </view>
    </view>

    <!-- 返回顶部按钮 -->
    <view
      v-if="showBackTop"
      class="back-top"
      @click="scrollToTop"
    >
      <text class="back-top-text">↑</text>
    </view>
  </view>
</template>



<script>
import { tabs, newsList } from "../../data/news";
export default {
  data() {
    return {
      tabs,
      list: [...newsList],
      currentTab: 0,
      // 回到顶部按钮状态与动态显示阈值
      showBackTop: false,
      currentScrollTop: 0,
      backTopThreshold: 0,
    };
  },
  computed: {
    filteredList() {
      if (this.currentTab === 0) return this.list;
      const category = this.tabs[this.currentTab];
      return this.list.filter((item) => item.category === category);
    }
  },
  onReady() {
    this.updateBackTopThreshold();
  },
  onPageScroll(event) {
    // 根据第一条新闻是否完全离开屏幕，决定是否显示回到顶部按钮
    this.currentScrollTop = event.scrollTop;
    this.showBackTop = this.backTopThreshold > 0 && event.scrollTop >= this.backTopThreshold;
  },

  // 这里代码截图截断，后续方法需要补全
  methods: {
  // Tab分类切换方法
  changeTab(index) {
    // 重复点击当前Tab，直接终止执行，避免无效刷新
    if (this.currentTab === index) return;

    // 判断当前是否有滚动距离，后续需要回到顶部
    const shouldScrollTop = this.currentScrollTop > 0;
    // 更新当前选中Tab下标
    this.currentTab = index;
    // 切换分类时，先隐藏返回顶部按钮
    this.showBackTop = false;
    // 重置滚动记录值
    this.currentScrollTop = 0;

    // 等待DOM更新完成后，再执行滚动回顶
    this.$nextTick(() => {
      if (shouldScrollTop) {
        uni.pageScrollTo({
          scrollTop: 0,
          duration: 0, // 无动画，瞬间置顶，避免卡顿
          fail: () => {}, // 空失败回调，屏蔽平台报错
        });
      }
    });

    // 延迟测量新列表卡片位置，更新返回顶部触发阈值
    setTimeout(() => {
      this.updateBackTopThreshold();
    }, 60);
  },

removeItem(index) {
  // 分类页删除的是过滤后的列表项，需要回到原始 list 里找到对应 id
  const category = this.tabs[this.currentTab];

  // 全部分类页：直接按下标删除原始数据
  if (this.currentTab === 0) {
    this.list.splice(index, 1);
  } else {
    // 其他分类页：先拿到筛选后的目标条目
    const target = this.filteredList[index];
    // 在完整原始列表中，通过唯一id找到真实下标
    const i = this.list.findIndex((item) => item.id === target.id);
    // 找到对应数据再删除
    if (i !== -1) this.list.splice(i, 1);
  }

  // 删除后重新计算返回顶部触发阈值
  this.updateBackTopThreshold();
},

  // 动态计算返回顶部按钮触发阈值
  updateBackTopThreshold() {
    // 等待DOM渲染完毕，再进行节点测量
    this.$nextTick(() => {
      // 创建UniApp节点查询器，绑定当前页面组件
      const query = uni.createSelectorQuery().in(this);
      // 选中第一个新闻卡片.card元素
      query
        .select(".card")
        // 获取节点尺寸、位置信息
        .boundingClientRect((rect) => {
          // 节点不存在/未渲染，重置阈值
          if (!rect) {
            this.backTopThreshold = 0;
            this.showBackTop = false;
            return;
          }
          // 计算精准触发高度：卡片完全滑出屏幕时才显示返回顶部
          this.backTopThreshold = rect.top + this.currentScrollTop + rect.height;
        })
        .exec(); // 执行查询
    });
  },

  // 一键滚动回到顶部
  scrollToTop() {
    uni.pageScrollTo({
      scrollTop: 0,
      duration: 500
    });
  },

  // 跳转到新闻详情页
  goDetail(id) {
    uni.navigateTo({
      url: `/pages/detail/detail?id=` + id,
    });
  },
},

};
</script>


<style >
	.page {
	    background-color: #f5f5f5;
	}
	
	/* Tab */
	.tab-bar {
	    display: flex;
	    align-items: stretch;
	    background-color: #ffffff;
	    border-bottom: 1rpx solid #eeeeee;
	    flex-shrink: 0;
	}
	
	.tab-item {
	    flex: 1;
	    display: flex;
	    align-items: center;
	    justify-content: center;
	    height: 88rpx;
	    font-size: 28rpx;
	    color: #666666;
	    border-bottom: 4rpx solid transparent;
	    box-sizing: border-box;
	}
	
	.tab-item.active {
	    color: #4A90D9;
	    border-bottom-color: #4A90D9;
	    font-weight: bold;
	}
	
	/* 卡片 */
	.card {
	    display: flex;
	    align-items: center;
	    background-color: #ffffff;
	    margin: 16rpx 24rpx;
	    border-radius: 12rpx;
	    padding: 18rpx;
	    position: relative;
	}
	
	.cover {
	    width: 200rpx;
	    height: 140rpx;
	    border-radius: 8rpx;
	    flex-shrink: 0;
	}
	
	.info {
	    flex: 1;
	    padding: 0 20rpx;
	    display: flex;
	    flex-direction: column;
	    justify-content: space-between;
	    height: 140rpx;
	}
	
	.title {
	    font-size: 28rpx;
	    color: #333333;
	    line-height: 1.5;
	    display: -webkit-box;
	    -webkit-line-clamp: 3;
	    -webkit-box-orient: vertical;
	    overflow: hidden;
	}
	
	.meta {
	    display: flex;
	    gap: 16rpx;
	}
	
	.meta-text {
	    font-size: 22rpx;
	    color: #999999;
	}
	
	/* 关闭按钮 */
	.close {
	    position: absolute;
	    top: 12rpx;
	    right: 0;
	    font-size: 36rpx;
	    color: #cccccc;
	    line-height: 1;
	}
	
	/* 空状态 */
	.empty {
	    text-align: center;
	    padding: 100rpx 0;
	    color: #cccccc;
	    font-size: 28rpx;
	}
	
	.back-top {
	    position: fixed;
	    right: 30rpx;
	    bottom: calc(128rpx + env(safe-area-inset-bottom));
	    width: 72rpx;
	    height: 72rpx;
	    border-radius: 50%;
	    background-color: rgba(255, 255, 255, 0.92);
	    border: 1rpx solid rgba(74, 144, 217, 0.22);
	    box-shadow: 0 10rpx 24rpx rgba(43, 92, 139, 0.12);
	    display: flex;
	    align-items: center;
	    justify-content: center;
	    z-index: 20;
	}
	
	.back-top:active {
	    background-color: #f3f8fd;
	    transform: translateY(2rpx);
	}
	
	.back-top-arrow {
	    color: #4A90D9;
	    font-size: 34rpx;
	    font-weight: 600;
	    line-height: 1;
	}
</style>
