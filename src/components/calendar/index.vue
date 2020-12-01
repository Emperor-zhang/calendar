<template>
  <view class="content">
    <view class="mycha">
      <view class="thead">
        <view class="dateLeft">
          <image
            class="lastMonth"
            src="../../static/prev.png"
            @click="lastMonth"
            mode="widthFix"
          ></image>
          <view class="nowDate">{{ year1 }}年{{ month }}月</view>
          <image
            class="nextMonth"
            src="../../static/next.png"
            @click="nextMonth"
            mode="widthFix"
          ></image>
        </view>
        <view class="clearSelected" @click="clearSelected">清除选择</view>
        <view class="backNow" @click="backNow">回到今天</view>
      </view>

      <view class="calendarwa">
        <view class="secd">
          <view>一</view>
          <view>二</view>
          <view>三</view>
          <view>四</view>
          <view>五</view>
          <view>六</view>
          <view>日</view>
        </view>
        <view class="dateList">
          <view
            class="dateBox"
            v-for="(item, index) in days"
            :key="index"
            :class="{
              active: selected.includes(item.time),
              borderColor: isActive == item.time,
            }"
            @click="selectDate(item.time)"
            >{{ item.content }}<view>{{ item.pre }}</view></view
          >
        </view>
      </view>
    </view>
  </view>
</template>
<script>
import { getResquest } from "@/utils/api.js";
var that;
export default {
  data() {
    return {
      year: null, //年份
      year1: null,
      month: null, //月份
      date: null, //日
      days: [], //当月的日期列表
      arr: [], //所有日期转换时间戳
      arr1: [], //把数据库日历添加到这里面-------------------这里是年月日  下面的方法中需要 把日遍历出来重新给arr1赋值  需要对应月份
      daynum: [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31], //1-12月每个月的天数
      obj: {
        pre: "x",
        content: "1",
        time: "",
      },
      isActive: new Date(),
      selected: [],
      sendTime: "",
      lastCount: 0,
      mapList: [],
      count: 0,
      overArr: [],
    };
  },
  mounted() {},

  created() {
    that = this;
    let time = new Date();
    that.year = time.getFullYear();
    that.year1 = time.getFullYear();
    that.date = time.getDate();
    that.month =
      (Math.abs(time.getMonth()) < 10
        ? "0" + Math.abs(time.getMonth())
        : time.getMonth()) + 1;
    that.isyear(that.year); //判断平年闰年
    that.getPort();
    that.isActive = that.timestamp(
      that.year1 + "-" + that.month + "-" + that.date
    );
    console.log(that.isActive);
  },
  methods: {
    getPort() {
      getResquest("/ticket/querySurplusTicket", {
        classesId: "5447",
        uuid: "33528117",
        accessToken: "cd6d1e38-d1c0-42e8-8857-fbb045fb1689",
      }).then((res) => {
        that.mapList = res.data.obj.mapList;
        // 把请求到的放票日期遍历出来变转化成时间戳
        that.arr1 = that.mapList.map((res) => {
          return that.timestamp(res.sendTime);
        });

        that.nowmonth();
      });
    },
    // 转换时间戳
    timestamp(sendTime) {
      return (
        new Date(sendTime.substring(0, 10) + " 00:00:00").valueOf() ||
        new Date(
          sendTime.substring(0, 10).replace(/-/g, "/") + " 00:00:00"
        ).valueOf()
      );
    },
    // 判断平年闰年
    isyear(val) {
      if (
        (val % 4 == 0 && val % 100 != 0) ||
        (val % 100 == 0 && val % 400 == 0)
      ) {
        that.daynum[1] = 29;
      } else {
        that.daynum[1] = 28;
      }
    },
    // 计算当月多少天
    nowmonth() {
      var daylength = that.daynum[that.month - 1];
      that.days.splice(0); //清空原数组
      for (let i = 0; i < daylength; i++) {
        that.days.push(
          (that.obj = {
            pre: "",
            content: i + 1,
            time: that.timestamp(
              (
                that.year1 +
                "-" +
                that.month +
                "-" +
                (i < 9 ? "0" + (i + 1) : i + 1)
              ).valueOf()
            ),
          })
        ); //根据获取的天数向数组中添加日期
      }
      that.getTicket();
      //获取当月1号是周几，然后向数组中添加空字符串占位
      var times = new Date(that.year1, that.month - 1, 1).getDay();
      if (times == 0) {
        //如果times是0则在前面添加6个空字符（视情况，有的星期天排在最前面）
        times = 6;
      } else {
        times--; //返回6则添加5个空字符，以此内推，返回1则不添加
      }
      for (let f = 0; f < times; f++) {
        that.days.unshift(" ");
      }
    },
    // 选择日期----可多选
    selectDate(data) {
      if (that.selected.indexOf(data) > -1) {
        // 判断数组中是否有当前字符串，有的话说明已经被选中了  再点击的时候就删除此字符串  取消选中
        that.selected.splice(
          that.selected.findIndex((item) => item === data),
          1
        );
      } else {
        if (that.arr1.indexOf(data) > -1) {
          that.selected.push(data);
        } else {
          uni.showToast({
            title: "请选择有效日期",
            image: "../../static/err.png",
            mask: true,
          });
        }
      }
    },
    // 清除选择
    clearSelected() {
      that.selected = [""];
    },
    // 点击购票
    buyTicket() {
      that.selected.filter((res) => {
        if (that.arr1.indexOf(res) > -1) {
          let index = that.arr1.indexOf(res);
          let shiftId = that.mapList[index].shiftId;
          that.count++;
          that.overArr.push({ count: that.count, shiftId: shiftId });
          //that.overArr 需要上传接口的对象型数组
        }
      });
    },
    // 显示票数
    getTicket() {
      that.arr = that.days.map((res) => {
        let d =
          that.year1 +
          "-" +
          that.month +
          "-" +
          (res.content < 10 ? "0" + res.content : res.content);
        return that.timestamp(d);
      });

      that.arr1.filter((res) => {
        if (that.arr.indexOf(res) > -1) {
          let index = that.arr.indexOf(res);
          let num = that.arr1.indexOf(that.arr[index]);
          let lastCount = that.mapList[num].lastCount;
          let val = "余票:" + lastCount;
          if (lastCount == -1) {
            val = "停止售票";
          }
          return (that.days[index].pre = val); //"res"放置动态票数数据
        }
      });
    },
    //回到今天
    backNow() {
      let date = new Date();
      that.year1 = new Date().getFullYear();
      that.month =
        (Math.abs(date.getMonth()) < 10
          ? "0" + Math.abs(date.getMonth())
          : date.getMonth()) + 1;
      that.nowmonth();
    },
    // 上个月
    lastMonth() {
      if (that.month == 1) {
        that.month = 12;
        that.year1--;
        that.isyear(that.year1);
      } else {
        that.month--;
      }
      that.nowmonth();
    },
    // 下个月
    nextMonth() {
      if (that.month == 12) {
        that.month = 1;
        that.year1++;
        that.isyear(that.year1);
      } else {
        that.month++;
      }
      that.nowmonth();
    },
  },
};
</script>
<style lang="scss" scoped>
.active {
  background-color: #cc845a;
  color: #fff;
}
.borderColor {
  border: 1px solid #b66d6d !important;
  border-radius: 8rpx;
}
.content {
  position: relative;
  margin: 0 auto;
  width: 100%;
  height: 700rpx;
  .mycha {
    width: 100%;
    height: 100%;
    .thead {
      width: 100%;
      height: 80rpx;
      display: flex;
      align-items: center;
      justify-content: space-around;
      background-color: cornflowerblue;
      .dateLeft {
        width: 400rpx;
        display: flex;
        align-items: center;
        justify-content: space-around;
        height: 80rpx;
        .lastMonth {
          width: 40rpx;
        }
        .nowDate {
          width: 200rpx;
          background-color: rgb(136, 99, 148);
          display: flex;
          align-items: center;
          justify-content: center;
          border-radius: 10rpx;
          color: #fff;
        }
        .nextMonth {
          width: 40rpx;
        }
      }
      .clearSelected {
        width: 140rpx;
        height: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 24rpx;
        background-color: rgb(136, 99, 148);
        border-radius: 50rpx;
        color: #fff;
      }
      .backNow {
        width: 140rpx;
        height: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 24rpx;
        background-color: rgb(136, 99, 148);
        border-radius: 50rpx;
        color: #fff;
      }
    }
    .calendarwa {
      width: 100%;
      height: 520rpx;
      .secd {
        width: 100%;
        height: 50rpx;
        font-size: 26rpx;
        display: flex;
        align-items: center;
        justify-content: flex-start;
        background-color: rgb(74, 86, 107);
        color: #fff;
        view {
          width: 13.5%;
          height: 50rpx;
          display: flex;
          align-items: center;
          justify-content: center;
        }
      }
      .dateList {
        width: 100%;
        height: 470rpx;
        display: flex;
        align-items: flex-start;
        justify-content: flex-start;
        flex-wrap: wrap;
        margin-top: 20rpx;
        .dateBox {
          width: 13%;
          height: 80rpx;
          font-size: 30rpx;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: flex-start;
          margin-bottom: 8rpx;
          border: 1px solid #fff;
          view {
            font-size: 20rpx;
            color: #ff0000;
          }
        }
      }
    }
  }
}
</style>
