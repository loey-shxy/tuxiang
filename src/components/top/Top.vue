<template>
    <view :class="['top', textLeft && 'text-left']" :style="{ background: backgroundColor }">
            <view class="back-icon" @click="navigateBack">
                <u-icon :color="backIconColor" bold size="24" name="arrow-left"></u-icon>
            </view>
            <text class="title text-ellipsis" v-if="title" style="line-height: 20px">{{ title }}</text>
    </view>
</template>
<script>
export default {
    props: {
        backgroundColor: {
            type: String,
            default: 'linear-gradient(180deg, #A1F5E2 0%, #EAF6F5 100%);'
        },
        backIconColor: {
            type: String,
            default: '#006848'
        },
        title: {
            type: String,
            default: ''
        },
        textLeft: {
            type: Boolean,
            default: false
        }
    }
}
</script>
<style lang="scss" scoped>
.top {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    padding: 120px 30px 30px 30px;
    text-align: center;
    z-index: 99;
    .back-icon {
        position: absolute;
        left: 30px;
        top: 120px;
    }
    .title {
        display: inline-block;
        max-width: 80%;
    }
    &.text-left {
        text-align: left;
        .title {
            margin-left: 60px;
        }
    }
}
</style>