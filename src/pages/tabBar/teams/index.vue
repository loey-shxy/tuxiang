<template>
  <view class="app-container">
        <!-- <tab-bar :current="2"></tab-bar> -->
    <view class="app-top-background home"></view>
    <view class="page-title">
      <text>团队</text>
    </view>
    <search @confirm="searchConfirm" />

    <view class="menu-wrapper">
        <u-row gutter="10">
            <u-col span="6" v-for="(m, index) in menus" :key="index">
                <navigator :url="m.url" hover-class="navigator-hover-class">
                    <view :class="['menu-item', `menu-item__${index+1}`]">
                        <view class="icon-box">
                            <cover-image :src="m.icon"></cover-image>
                        </view>
                        <view><text>{{ m.title }}</text></view>
                    </view>
                </navigator>
            </u-col>
        </u-row>
    </view>

    <view class="teams-group">
        <view class="team-item" v-for="(item, index) in teams" :key="index">
            <view class="avatar">
              <image :src="item.logo" mode="aspectFill"></image>
            </view>
            <view class="name text-ellipsis"><text>{{ item.title }}</text></view>
            <view class="desc text-ellipsis">
              <u-icon name="file-text" size="16" v-if="item.note" ></u-icon>
              <text>{{ item.note }}</text>
            </view>
            <view class="member-num">{{ item.zrs }}</view>
            <view class="members">团队人数</view>
            <view class="date"><text>{{ moment(item.createTime).format('YYYY年MM月DD日') }}</text></view>
            <view class="button" v-if="userInfo.id != item.dzid">
                <u-button type="primary" shape="circle" text="申请加入" @click="joinTeam(item)"></u-button>
            </view>
        </view>
    </view>
    <u-modal
    :show="showPwd" 
    title="入团密码"
    closeOnClickOverlay
    showCancelButton
    @confirm="addMember"
    @cancel="close"
    @close="close">
        <view class="modal-content">
            <u-input type="password" v-model="addParams.pwd" border="bottom" />
        </view>
    </u-modal>
  </view>
</template>
<script>
import Search from '@/components/pageSearch/PageSearch';
import moment from 'moment'
import { mapGetters } from 'vuex';
export default {
  components: {
    Search,
  },
  data() {
    return {
      showPwd: false,
      menus: [
        {
          icon: '//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/wodetuandui.png',
          title: '我的团队',
          url: '/teamsPages/pages/myTeam/index',
        },
        {
          icon: '//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/chuangjiantuandui.png',
          title: '创建团队',
          url: '/teamsPages/pages/createTeam/index',
        }
        // {
        //   icon: '//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/store.png',
        //   title: '逛商场',
        //   url: '/pages/mall/index',
        // },
      ],
      teams: [],
      total: 0,
      query: {
        pageNum: 1,
        pageSize: 10,
        title: '',
        dzxm: '',
        dzsjh: ''
      },
      addParams: {
        teamCode: null,
        pwd: null,
        activityId: null
      }
    }
  },
  onShow() {
    this.teams = []
    this.getTeams()
  },
  onReachBottom() {
    if (this.teams.length === this.total) {
      return
    }
    this.query.pageNum++
    this.getTeams()
  },
  onPullDownRefresh() {
    wx.stopPullDownRefresh();
    setTimeout(() => {
      this.query.pageNum = 1
      this.getTeams()
    }, 500)
  },
  computed: {
    ...mapGetters(['userInfo'])
  },
  methods: {
    moment,
    searchConfirm(search) {
      this.query.title = search
      this.query.pageNum = 1
      this.teams = []
      this.getTeams()
    },
    getTeams() {
      if (this.query.pageNum === 1) {
        this.teams = []
      }
      this.$api.teamList(this.query).then(res => {
        this.teams = [...this.teams, ...res.rows]
        this.total = res.total
      })
    },
    addMember() {
      this.$api.joinTeam({
        ...this.addParams,
        openid: uni.getStorageSync('openid')
      }).then(res => {
        uni.showToast({
          icon: 'none',
          title: '你已成功加入团队',
          mask: true,
          duration: 1000
        })
        setTimeout(() => {
          this.query.pageNum = 1
          this.getTeams()
          this.showPwd = false
        }, 1000)
      })
    },
    joinTeam(team) {
      this.addParams = {
        ...this.addParams,
        teamCode: team.teamCode
      }
      if (team.jdyq == 0) {
        this.addMember()
        return
      }
      if (team.jdyq == 1) {
        this.showPwd = true
        return
      }
      uni.showToast({
        icon: 'none',
        title: '仅限团队邀请',
        mask: true,
        duration: 1000
      })
    },
    close() {
      this.showPwd = false
      this.addParams.pwd = null
    }
  }
};
</script>
<style lang="scss" scoped>
.menu-wrapper {
  margin-top: 60px;
  /deep/ .u-row {
    justify-content: center !important;
  }
  .menu-item {
    width: 210px;
    height: 142px;
    border-radius: 30px;
    text-align: center;
    backdrop-filter: blur(10px);
    border: 1px solid transparent;
    margin: 0 auto;
    .icon-box {
      margin: 0 auto;
      margin-top: 20px;
    }
    text {
      color: #fff;
      font-size: 30px;
      line-height: 42px;
      margin-top: 10px;
    }
    &.menu-item__1 {
      background: #b4c5fa;
      box-shadow: 0px 17px 37px 0px #b4c5fa;
      .icon-box {
        width: 60px;
        height: 50px;
      }
    }
    &.menu-item__2 {
      background: #6ebafa;
      box-shadow: 0px 17px 37px 0px #6ebafa;
      .icon-box {
        width: 62px;
        height: 50px;
      }
    }
    &.menu-item__3 {
      background: #ff992a;
      box-shadow: 0px 17px 37px 0px #ff992a;
      .icon-box {
        width: 56px;
        height: 50px;
      }
    }
  }
}
.teams-group {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  .team-item {
    text-align: center;
    margin-top: 90px;
    // height: 525px;
    padding-bottom: 50px;
    background: linear-gradient(335deg, rgba(255, 255, 255, 0) 0%, #fff 100%);
    box-shadow: 0px 17px 23px 0px rgba(138, 131, 168, 0.1);
    border-radius: 30px;
    width: 48%;
    .avatar {
      width: 126px;
      height: 126px;
      border: 7px solid #fff;
      box-shadow: 0px 2px 12px 0px rgba(0, 0, 0, 0.1);
      border-radius: 50%;
      margin: 0 auto;
      transform: translateY(-43px);
      overflow: hidden;
      image {
        width: 100%;
        height: 100%;
        border-radius: 50%;
      }
    }
    .name {
      color: #2a2a2a;
      font-size: 34px;
      line-height: 44px;
      font-weight: bold;
    }
    .desc {
      color: #868686;
      font-size: 28px;
      line-height: 44px;
      height: 44px;
      margin-top: 8px;
      padding: 0 6px;
      /deep/ .u-icon {
        display: inline-block;
        vertical-align: -4px;
      }
    }
    .member-num {
      font-size: 36px;
      line-height: 42px;
      color: #24af84;
      font-weight: bold;
      margin-top: 10px;
    }
    .members {
      font-weight: 300;
      color: #c2bcbc;
      line-height: 37px;
      font-size: 26px;
      margin-top: 2px;
    }
    .date {
      width: 85%;
      margin: 22px auto auto;
      border-top: 1px dashed #d7d7d7;
      border-bottom: 1px dashed #d7d7d7;
      color: #c2bcbc;
      line-height: 46px;
      font-size: 26px;
    }
    .button {
      width: 213px;
      margin: 32px auto auto;
      /deep/ .u-button--normal {
        font-size: 32px;
        height: 55px;
      }
    }
  }
}
.modal-content {
  /deep/ .u-button {
    margin-top: 30px;
  }
  /deep/ .u-border-bottom {
      border-bottom: 1px solid #e2e2e2;
  }
}
.button__wrapper {
  display: flex;
  justify-content: space-between;
}
</style>
