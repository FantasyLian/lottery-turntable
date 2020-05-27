<template>
  	<div class="container">
    	<div class="lottery-turntable">
      		<div class="lottery-title"></div>
      		<div class="lottery-main">
        		<div class="lottery-pointer" @click="beginRotate()"></div>
        		<div class="lottery-bg" :style="rotateStyle">
					<div class="prize-list">
						<div
							class="prize-item"
							v-for="(item,index) in prizeList"
							:key="index"
							:style="item.style"
						>
							<div class="prize-pic">
								<img :src="item.icon" />
							</div>
              				<div class="prize-type">{{item.name}}</div>
           				</div>
          			</div>
       			</div>
      		</div>
    	</div>
		<div class="main">
			<div class="main-bg"></div>
			<div class="bg-p"></div>
			<div class="content">
				<div class="count">今日免费抽奖次数： {{ count }}</div>
			</div>
			<div class="tip">
				<div class="tip-title">活动规则</div>
				<div class="tip-content">
					<p>1.每日签到后，即可获得一次幸运大转盘的机会。</p>
					<p>2.金币抽奖，每10个金豆可以兑换一次大转盘抽奖机会</p>
				</div>
			</div>
		</div>

		<!-- 中奖弹框 -->
		<div class="toast" v-show="prize">
			<div class="toast-container">
				<img :src="toastIcon" class="toast-picture" />
				<div class="close" @click="closeToast()"></div>
				<div class="toast-title">{{toastTitle}}</div>
				<div class="toast-btn">
					<div class="toast-cancel" @click="closeToast">关闭</div>
				</div>
			</div>
		</div>
		<div class="toast-mask" v-show="prize"></div>
  	</div>
</template>

<script>
import { prizeList } from "./config";
const CIRCLE_ANGLE = 360;

const config = {
	// 总旋转时间
	duration: 4000,
	// 旋转圈数
	circle: 8,
	mode: "ease-in-out"
};

export default {
	data() {
		return {
			count: 10, // 剩余抽奖次数
			duration: 10000, // 转盘旋转时间
			prizeList: [], // 奖品列表
			rotateAngle: 0, // 旋转角度
			index: 0,
			prize: null
		}
	},
	created() {
		// 初始化一些值
		this.angleList = [];
		// 是否正在旋转
		this.isRotating = false;
		// 基本配置
		this.config = config;
		// 获取奖品列表
		this.initPrizeList();
	},
	computed: {
		rotateStyle() {
			return `
				-webkit-transition: transform ${this.config.duration}ms ${this.config.mode};
				transition: transform ${this.config.duration}ms ${this.config.mode};
				-webkit-transform: rotate(${this.rotateAngle}deg);
					transform: rotate(${this.rotateAngle}deg);`;
		},
		// 抽奖结果弹框标题
		toastTitle() {
			return this.prize && this.prize.isPrize === 1 ? "恭喜您，获得" + this.prize.name : "未中奖";
		},
		// 抽奖结果弹框图标
		toastIcon() {
			return this.prize && this.prize.isPrize === 1 ? require("../assets/img/congratulation.png") : require("../assets/img/sorry.png");
		}
	},
	methods: {
		initPrizeList() {
			// 这里可以发起请求，从服务端获取奖品列表
			// 这里使用模拟数据
			this.prizeList = this.formatPrizeList(prizeList);
		},
		// 格式化奖品列表，计算每个奖品的位置
		formatPrizeList(list) {
			// 记录每个奖的位置
			const angleList = [];

			const l = list.length;
			// 计算单个奖项所占的角度
			const average = CIRCLE_ANGLE / l;

			const half = average / 2;

			// 循环计算给每个奖项添加style属性
			list.forEach((item, i) => {
				// 每个奖项旋转的位置为 当前 i * 平均值 + 平均值 / 2
				const angle = -(i * average + half);
				// 增加 style
				item.style = `-webkit-transform: rotate(${angle}deg);
							transform: rotate(${angle}deg);`;

				// 记录每个奖项的角度范围
				angleList.push(i * average + half);
			});

			this.angleList = angleList;

			return list;
		},

		beginRotate() {
			// 添加次数校验

			if (this.count === 0) return;

			// 开始抽奖
			// 这里这里向服务端发起请求，得到要获得的奖
			// 可以返回下标，也可以返回奖品 id，通过查询 奖品列表，最终得到下标

			// 随机获取下标
			this.index = this.random(this.prizeList.length - 1);

			// 减少剩余抽奖次数
			this.count--;

			// 开始旋转
			this.rotating();
		},

		random(max, min = 0) {
			return parseInt(Math.random() * (max - min + 1) + min);
		},

		rotating() {
			const { isRotating, angleList, config, rotateAngle, index } = this;

			if (isRotating) return;

			this.isRotating = true;

			// 计算角度
			const angle =
				// 初始角度
				rotateAngle +
				// 多旋转的圈数
				config.circle * CIRCLE_ANGLE +
				// 奖项的角度
				angleList[index] -
				(rotateAngle % CIRCLE_ANGLE);

			this.rotateAngle = angle;

			// 旋转结束后，允许再次触发
			setTimeout(() => {
				this.rotateOver();
			}, config.duration + 1000);
		},

		rotateOver() {
			this.isRotating = false;

			this.prize = prizeList[this.index];

			console.log(this.prize, this.index);
		},

		//关闭弹窗
		closeToast() {
			this.prize = null;
		}
	}
};
</script>

<style lang="css">
	@import 'style.css';
</style>