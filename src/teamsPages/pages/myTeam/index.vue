<template>
  <view class="app-container">
    <!-- 页面标题栏 -->
    <view class="top-title">
      <view class="title-content">
        <!-- 返回 -->
        <view class="arrow-back">
            <u-icon @click="navigateBack" bold color="#006848" name="arrow-left" size="26"></u-icon>
        </view>
        <text>我的团队</text>
      </view>
    </view>
    <!-- 顶部背景图 -->
    <view class="app-top-background">
      <image class="image" src="//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/myteam_top_bg.png"></image>
    </view>

    <view class="team-wrapper">
      <view class="team-list" v-for="team in teams" :key="team.id">
        <!-- 顶部内容版块 -->
        <view class="top-wrapper">
          <!-- 团队信息 -->
          <view class="info">
            <view class="info-name">
              <text class="name text-ellipsis">{{ team.title }}</text>
              <view class="team-btn">
                <button 
                open-type="share" 
                id="invite_team" 
                :data-team-code="team.teamCode"
                :data-image="team.logo">邀请</button>
                <button @click="checkIn(team)" :class="[hasCheckIn && 'disabled']">{{ hasCheckIn?'已签':'签到' }}</button>
                <button @click="quitTeam(team)">退团</button>
              </view>
            </view>

            <!-- 头像 -->
            <view class="info-avatar">
              <view class="avatar">
                <image class="image" :src="team.logo" mode="aspectFill"></image>
              </view>
              <view class="right">
                <view>
                  <view class="name row-box position">
                    <text>职务</text>
                    <!-- <text></text> -->
                    <text>队长</text>
                  </view>
                  <view class="role row-box position">
                    <text>{{ userInfo.id == team.dzid ? '队长' : '成员' }}</text>
                    <!-- <text>副队长</text> -->
                    <text>{{ team.dzxm }}</text>
                  </view>
                </view>
                <view class="data-box">
                  <view class="row-box">
                    <text class="num">{{ team.zrs }}</text>
                    <text class="num">{{ team.zjf }}</text>
                  </view>
                  <view class="row-box">
                    <text>总人数</text>
                    <text>积分</text>
                    <navigator v-if="team.dzid == userInfo.id" :url="`/teamsPages/pages/createTeam/index?id=${team.id}`" hover-class="navigator-hover-class">
                      <view class="info-edit">
                        信息编辑
                      </view>
                    </navigator>
                  </view>
                </view>
              </view>
            </view>

            <!-- 简介 -->
            <view class="info-introduce">
              <text>{{ team.note }}</text>
            </view>
          </view>
        <!-- </view> -->

          <!-- 功能区 -->
          <view class="feature-wrapper">
            <view class="feature-item__content" v-for="(item, index) in features(team)" :key="index">
              <navigator :url="item.option ? `${item.url}?teamId=${team.id}&dzid=${team.dzid}` : item.url" hover-class="navigator-hover-class">
                  <view class="feature-item">
                    <image class="icon-image" :src="item.icon"> </image>
                    <text>{{ item.name }}</text>
                  </view>
                </navigator>
            </view>
          </view>
        </view>

        <!-- 活动列表 -->
        <view class="activity-group">
          <view class="activity-item" v-for="(item, index) in team.activities" :key="index">
            <navigator 
            hover-class="navigator-hover-class" 
            :url="`/productPages/pages/productDetail/index?cpbh=${item.cpbh}&teamId=${team.id}&activityId=${item.id}&activityDate=${item.beginTime}`">
              <view class="activity-content">
                  <view class="left">
                    <image class="image" mode="aspectFill" :src="item.image"></image>
                  </view>
                  <view class="right">
                    <view class="title ellipsis-column-2"><text>{{ item.name }}</text></view>
                    <!-- <view class="place"><text>活动地点：</text><text class="place-name">{{ item.address }}</text></view> -->
                    <view class="date-box">
                      <view>活动时间：</view>
                      <view class="date">
                        <view class="date1">{{ item.beginTime }}</view>
                        <view class="date2">{{ item.finishTime }}</view>
                      </view>
                    </view>
                  </view>
              </view>
            </navigator>
            <view class="btn-wrapper">
              <view class="button">
                <button 
                shape="circle" 
                color="#17aa7d" 
                open-type="share" 
                id="share_activity" 
                :data-activity-id="item.id"
                :data-team-code="team.teamCode"
                :data-image="item.image">邀请好友</button>
              </view>
              <view class="button">
                <u-button shape="circle" color="#74b9fd" @click="toSignUp(item.id)">报名列表</u-button>
              </view>
              <view class="button">
                <template v-if="item.isbm == 0">
                  <u-button 
                  shape="circle" 
                  v-if="moment().isBefore(item.deadlineTime)" 
                  color="#f8ab52" 
                  @click="joinActivity(team.id, item.id)">立即报名</u-button>
                  <u-button shape="circle" v-else color="#f8ab52">报名结束</u-button>
                </template>
                <u-button v-else-if="item.isbm == 1" shape="circle" color="#f8ab52" @click="cancel(team, item)">取消报名</u-button>
                <u-button v-else-if="item.isbm == 2" shape="circle" color="#f8ab52">报名成功</u-button>
              </view>
            </view>
            <view v-if="index < team.activities.length-1">
              <image class="decor left-decor"
                src="//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/myteam_activity_decor.png"></image>
              <image class="decor right-decor"
                src="//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/myteam_activity_decor.png"></image>
            </view>
          </view>
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
            <u-input type="password" v-model="joinParams.pwd" border="bottom" />
        </view>
    </u-modal>
  </view>
</template>

<script>
import { mapGetters } from 'vuex'
import moment from 'moment'
export default {
  data() {
    return {
      showPwd: false,
      team: {},
      teams: [],
      defaultTeam: {},
      userInfo: {},
      hasCheckIn: false,
      joinParams: {
        teamCode: null,
        activityId: null,
        pwd: null
      }
    };
  },
  onShow() {
    this.getMyTeam()
    let info = uni.getStorageSync('userInfo')
    this.userInfo = info ? JSON.parse(info) : {}
    this.getCheckInState()
  },
  onPullDownRefresh() {
    wx.stopPullDownRefresh();
    setTimeout(() => {
      this.getMyTeam()
    }, 500)
  },
  onLoad(options) {
    let { teamCode, activityId } = options
    teamCode = teamCode == null || teamCode == 'undefined' ? '' : teamCode
    activityId = activityId == null || activityId == 'undefined' ? '' : activityId
    
    this.joinParams.teamCode = teamCode
    this.joinParams.activityId = activityId
    
    if (teamCode && uni.getStorageSync('openid')) {
      this.getTeamInfoByCode(teamCode, activityId)
    }

    if (!uni.getStorageSync('openid')) {
      uni.navigateTo({ url: `/pages/login/index?teamCode=${teamCode}&activityId=${activityId}` })
    } 
  },
  methods: {
    moment,
    getTeamInfoByCode(teamCode, activityId) {
      this.$api.getTeamInfoByCode({ teamCode }).then(res => {
        const team = res.data
        if (team.jdyq == 0) {
          this.addMember()
          return
        }
        if (team.jdyq == 1) {
          this.showPwd = true
          return
        }
        if (activityId) {
          this.joinActivity(team.id, activityId)
        }
      })
    },
    addMember() {
      this.$api.joinTeam({
        ...this.joinParams,
        openid: uni.getStorageSync('openid')
      }).then(res => {
        uni.$u.toast('你已成功加入团队')
        uni.showToast({
          icon: 'none',
          title: '你已成功加入团队',
          mask: true,
          duration: 2000
        })
        setTimeout(() => {
          this.getMyTeam()
        }, 2000)
        this.showPwd = false
      })
    },
    close() {
      this.showPwd = false
      this.joinParams.pwd = null
    },
    features(team) {
      let actions = [
        {
          name: '发起活动',
          icon: '//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/activity.png',
          option: true,
          url: '/activityPages/pages/addActivity/index',
        },
        // {
        //   name: '排行榜',
        //   icon: '//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/rankings.png',
        //   url: '/productPages/pages/leaderboard/index',
        // },
        {
          name: '成员列表',
          icon: '//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/member.png',
          option: true,
          url: '/teamsPages/pages/member/index',
        },
        // {
        //   name: '服装商城',
        //   icon: '//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/mall.png',
        //   url: '/pages/mall/index',
        // }
      ]
      if (team.dzid != this.userInfo.id) {
        actions.shift()
      }

      return actions
    },
    toSignUp(activityId) {
      uni.navigateTo({ url: `/activityPages/pages/enroll/index?activityId=${activityId}` })
    },
    quitTeam(team) {
      let hasDis = 0
      if (this.userInfo.id == team.dzid) {
        hasDis = 1
        uni.showModal({
          title: '温馨提示',
          content: team.zrs > 1 ? '团队中尚有成员，是否继续解散团队？' : '确定解散团队吗？',
          success: res => {
            if (res.confirm) {
              this.submitQuit(team, '您已解散该团队', hasDis)
            }
          }
        })
      } else {
        uni.showModal({
          title: '温馨提示',
          content: '退团将清零您的积分，确定要退出团队吗？',
          success: res => {
            if (res.confirm) {
              this.submitQuit(team, '您已成功退出团队', hasDis)
            }
          }
        })
      }
    },
    submitQuit(team, msg, hasDis) {
      this.$api.quitTeam({
        loginUserId: this.userInfo.id,
        teamId: team.id,
        hasDis
      }).then(res => {
        uni.showToast({
          icon: 'none',
          title: msg,
          mask: true,
          duration: 2000
        })
        this.getMyTeam()
      })
    },
    joinActivitySubmit(teamId, activityId) {
      this.$api.addUserInActivity({
        activityId,
        teamId,
        openid: uni.getStorageSync('openid')
      }).then(res => {
        uni.showToast({
          icon: 'none',
          title: '已报名参加',
          mask: true,
          duration: 1000
        })
        setTimeout(() => {
          this.getActivity()
        }, 1000)
      })
    },
    joinActivity(teamId, activityId, showConfirm = true) {
      if (showConfirm) {
        uni.showModal({
          title: '温馨提示',
          content: '确定要报名参加此活动吗？',
          success: res => {
            if (res.confirm) {
              this.joinActivitySubmit(teamId, activityId)
            }
          }
        })
      } else {
        this.joinActivitySubmit(teamId, activityId)
      }
    },
    getMyTeam() {
      if (uni.getStorageSync('openid')) {
        this.$api.myTeamList({
          openid: uni.getStorageSync('openid')
        }).then(res => {
          if (res.data && res.data.length) {
            this.teams = res.data
            this.getActivity()
          } else {
            if (!this.joinParams.teamCode) {
              wx.showModal({
                title: '提示',
                content: '你暂未加入任何团队！',
                showCancel: false,
                success: res => {
                  if (res.confirm) {
                    uni.switchTab({ url: '/pages/tabBar/teams/index' })
                  }
                }
              })
            }
          }
        })
      }
    },
    getActivity() {
      this.teams.forEach(item => {
        this.$api.teamActivity({
          teamId: item.id,
          pageNum: 1,
          pageSize: 1000,
          uid: this.userInfo.id.toString(),
          userId: this.userInfo.id.toString()
        }).then(res => {
          this.$set(item, 'activities', res.rows)
        })
      })
    },
    cancel(team, activity) {
      uni.showModal({
          title: '温馨提示',
          content: '确定要取消报名吗？',
          success: res => {
              if (res.confirm) {
                this.$api.cancelActivity({
                  activityId: activity.id,
                  userId: this.userInfo.id
                }).then(res => {
                  uni.showToast({
                    icon: 'none',
                    title: '已取消报名此活动',
                    mask: true,
                    duration: 1000
                  })
                  setTimeout(() => {
                    this.getActivity()
                  }, 1000)
                })
              }
          }
      })
    },
    checkIn(team) {
      uni.navigateTo({ url: `/teamsPages/pages/checkIn/index?teamId=${team.id}&userId=${this.userInfo.id}` })
    },
    getCheckInState() {
      this.$api.hasCheckIn({
        userId: this.userInfo.id
      }).then(res => {
        this.hasCheckIn = res.data
      })
    }
  }
};
</script>

<style lang="scss" scoped>
.app-container {
  font-size: 28px;
  color: #a7afaf;
}

.app-top-background {
  height: 364px;
  margin: 0 -30px;
  position: absolute;
  z-index: -1;

  .image {
    width: 100%;
    height: 100%;
    overflow: hidden;
  }
}

.top-title {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  padding: 90px 0 30px;
  width: 100%;
  color: #006848;
  font-size: 36px;
  background: linear-gradient(to right, #b9f7e8 50%, #eaf6f5 100%);
  z-index: 1;

  .title-content {
    text-align: center;
  }

  .arrow-back {
    position: absolute;
    left: 30px;
    top: 90px;
  }
}

.team-wrapper {
  margin-top: 250px;
}

.top-wrapper {
  width: 690px;
  // height: 658px;
  border-radius: 20px;
  box-sizing: border-box;
  background: rgba(255, 255, 255, 0.68);
  box-shadow: 0px 17px 23px 0px rgba(138, 131, 168, 0.1);
  overflow: hidden;

  .info {
    padding: 50px 40px 20px 50px;

    .info-name {
      display: flex;
      flex-direction: space-between;

      .name {
        flex: 1;
        font-size: 38px;
        font-weight: bold;
        color: #2a2a2a;
      }

      .team-btn {
        button {
          display: inline-block;
          height: 66px;
          line-height: 60px;
          padding: 0 20px 0 25px;
          font-size: 28px;
          color: #f2fcfa;
          letter-spacing: 10px;
          text-align: center;
          border-radius: 39px;
          border: 1px solid #ffffff;

          &:nth-child(1) {
            background: rgba(255, 81, 42, 0.8);
            box-shadow: 0px 7px 25px 0px #ffcc94;
          }

          &:nth-child(2) {
            background: rgba(34, 96, 230, 0.8);
            box-shadow: 0px 7px 25px 0px #54adf6;
            margin-left: 20px;
            &.disabled {
              background: rgb(121, 159, 235);
            }
          }
          &:nth-child(3) {
            background: rgba(163, 158, 153, 0.8);
            box-shadow: 0px 7px 25px 0px #b3b1ac;
            margin-left: 20px;
          }
        }
      }
    }

    .info-gender {
      display: flex;
      flex-direction: row;
      align-items: center;

      .image {
        width: 30px;
        height: 30px;
        overflow: hidden;
        background-position: 0 0;
        background-size: 100% 100%;
        background-repeat: no-repeat;

        &.image-male {
          background-image: url('//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/male.png');
        }

        &.image-female {
          background-image: url('//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/icon/female.png');
          margin-left: 26px;
          margin-right: 58px;
        }
      }
    }

    .info-avatar {
      margin-top: 18px;
      display: flex;
      justify-content: center;
      align-items: center;

      .avatar {
        width: 190px;
        height: 190px;
        border-radius: 50%;
        overflow: hidden;

        image {
          border-radius: 50%;
        }
      }

      .right {
        flex: 1;
        padding-left: 52px;
      }

      .data-box {
        position: relative;
      }

      .row-box {
        line-height: 40px;

        &:last-child {
          margin-top: 10px;
        }

        text {
          display: inline-block;
          width: 30%;
          text-align: center;
        }

        .num {
          font-size: 36px;
          font-weight: bold;
          color: #17aa7d;
        }

        .info-edit {
          position: absolute;
          top: 50%;
          right: -40px;
          transform: translateY(-50%);
          width: 150px;
          height: 55px;
          line-height: 55px;
          font-size: 28px;
          font-weight: 600;
          color: #ffffff;
          text-align: center;
          background: #17aa7d;
          box-shadow: 0px 2px 4px 0px rgba(23, 170, 125, 0.6);
          border-radius: 100px 0px 0px 100px;
        }
        &.position text{
          overflow: hidden;
          white-space: nowrap;
          text-overflow: ellipsis;
          width: 50%;
        }
      }

      .name {
        color: #2a2a2a;
        font-weight: 600;
      }

      .role {
        margin: 10px 0 40px;
      }
    }

    .info-introduce {
      font-size: 26px;
      color: #a5a6ae;
      margin: 20px 0 10px;
      line-height: 42px;
    }
  }

  .feature-wrapper {
    height: 125px;
    padding: 20px 50px;
    background: rgba(227, 234, 232, 0.22);
    display: flex;
    justify-content: space-between;
    .feature-item__content {
      flex: 1;
    }

    .feature-item {
      display: flex;
      flex-direction: column;
      align-items: center;

      .icon-image {
        width: 30px;
        height: 30px;
        margin-bottom: 10px;
      }
    }
  }
}

.activity-group {
  .activity-item {
    position: relative;
    padding: 0 30px 75px;
    margin: 20px -23px 0;
    background-image: url('//mall-lyxcx.oss-cn-hangzhou.aliyuncs.com/front_end/myteam_activity_bg.png');
    background-position: 0 0;
    background-size: 100% 100%;
    background-repeat: no-repeat;

    .item-bg {
      position: absolute;
      width: 100%;
      height: 100%;
      left: 0;
      top: 0;
      z-index: -1;
    }

    .decor {
      position: absolute;
      width: 18px;
      height: 108px;
      bottom: -15px;

      &.left-decor {
        left: 35px;
      }

      &.right-decor {
        right: 35px;
      }
    }
  }

  .activity-content {
    display: flex;
    flex-direction: row;
    padding: 36px 40px 55px 30px;

    .left {
      margin-right: 25px;

      .image {
        width: 224px;
        height: 224px;
        border-radius: 16px;
        background: #d8d8d8;
        overflow: hidden;
      }
    }

    .right {
      font-size: 28px;
      color: #a7afaf;

      .title {
        color: #2a2a2a;
        font-size: 34px;
        font-weight: 600;
        margin-bottom: 30px;
      }

      .place-name {
        color: #2a2a2a;
      }

      .date-box {
        margin-top: 15px;
        display: flex;
        flex-direction: row;
        white-space: nowrap;

        .date {
          white-space: nowrap;
        }
      }
    }
  }

  .btn-wrapper {
    text-align: center;
    padding: 18px;
    display: flex;
    justify-content: space-between;

    .button {
      width: 190px;
      height: 55px;
      line-height: 55px;
      border-radius: 32px;
      font-size: 30px;
      color: #fff;
      /deep/ .u-button {
        height: 100%;
        width: 100%;
      }
      button {
        background-color: #17aa7d;
        color: #fff;
        width: 190px;
        height: 55px;
        line-height: 55px;
        border-radius: 32px;
        font-size: 30px;
      }
    }
  }
}
.button__wrapper {
  display: flex;
  justify-content: space-between;
}
</style>
