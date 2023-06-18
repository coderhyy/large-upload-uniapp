<template>
  <view class="">
    <view style="margin: 20rpx 0 0 20rpx">
      <u-upload
        multiple
        :maxCount="9"
        :maxSize="maxSize"
        @afterRead="afterRead"
      >
        <u-button color="#0ab99c" plain size="small" text="图片上传"></u-button>
      </u-upload>
    </view>
    <u-row>
      <u-col v-for="item in list" :key="item.id" span="6">
        <view style="margin: 20rpx">
          <u--image
            :showLoading="true"
            :src="imgURL + item.url"
            width="100%"
            height="100%"
            mode="widthFix"
          ></u--image>
          <u-link :href="imgURL + item.url" under-line text="复制链接"></u-link>
        </view>
      </u-col>
    </u-row>
    <u-loadmore :status="loadStatus" />
    <u-safe-bottom />
  </view>
</template>

<script>
  export default {
    data() {
      return {
        imgURL: 'https://www.never.vin/',
        baseURL: 'https://api.never.vin',
        maxSize: 1024 * 1024 * 2,
        queryInfo: {
          page: 1,
          pageSize: 10
        },
        list: [],
        loadStatus: 'loadmore'
      };
    },
    onLoad() {
      this.getList();
    },
    onPullDownRefresh() {
      try {
        this.queryInfo.page = 1;
        this.getList();
      } finally {
        uni.stopPullDownRefresh();
      }
    },
    onReachBottom() {
      if (this.loadStatus === 'nomore') return;
      this.queryInfo.page++;
      this.getList();
    },
    methods: {
      getList() {
        this.loadStatus = 'loading';
        uni.request({
          url: `${this.baseURL}/api/v1/file/list`,
          data: this.queryInfo,
          success: (res) => {
            if (this.queryInfo.page === 1) {
              this.list = res.data.data.list;
            } else {
              this.list = this.list.concat(res.data.data.list);
            }

            this.loadStatus = 'loadmore';
            if (this.list.length >= res.data.data.total) {
              this.loadStatus = 'nomore';
            }
          }
        });
      },

      afterRead(event) {
        console.log(event);
        const fileList = [...event.file];

        const promiseList = [];
        fileList.forEach(async (file) => {
          promiseList.push(this.uploadFilePromise(file.url));
        });

        Promise.all(promiseList).then((res) => {
          this.getList();
        });
      },

      uploadFilePromise(path) {
        return new Promise((resolve, reject) => {
          uni.uploadFile({
            url: `${this.baseURL}/api/v1/file/upload`,
            filePath: path,
            name: 'file',
            success(res) {
              resolve(JSON.parse(res.data));
            }
          });
        });
      }
    }
  };
</script>

<style scoped lang="scss">
  ::v-deep .u-row {
    flex-wrap: wrap;
  }
</style>
