<template>
	<view class="wxy-countdown">
		<text v-if="showDay" :style="{ borderColor: borderColor, color: color, backgroundColor: backgroundColor }" class="wxy-countdown__number">{{ d }}</text>
		<text v-if="showDay" :style="{ color: splitorColor }" class="wxy-countdown__splitor">天</text>
		<text :style="{ borderColor: borderColor, color: color, backgroundColor: backgroundColor }" class="wxy-countdown__number">{{ h }}</text>
		<text :style="{ color: splitorColor }" class="wxy-countdown__splitor">{{ showColon ? ':' : '时' }}</text>
		<text :style="{ borderColor: borderColor, color: color, backgroundColor: backgroundColor }" class="wxy-countdown__number">{{ i }}</text>
		<text :style="{ color: splitorColor }" class="wxy-countdown__splitor">{{ showColon ? ':' : '分' }}</text>
		<text :style="{ borderColor: borderColor, color: color, backgroundColor: backgroundColor }" class="wxy-countdown__number">{{ s }}</text>
		<text v-if="!showColon" :style="{ color: splitorColor }" class="wxy-countdown__splitor">秒</text>
	</view>
</template>
<script>
/**
 * Countdown 倒计时(基于官方增强修改版)
 * @description 倒计时组件
 * @tutorial https://ext.dcloud.net.cn/plugin?id=25
 * @property {String} backgroundColor 背景色
 * @property {String} color 文字颜色
 * @property {Number} day 天数
 * @property {Number} hour 小时
 * @property {Number} minute 分钟
 * @property {Number} second 秒
 * @property {Boolean} showDay = [true|false] 是否显示天数
 * @property {Boolean} showColon = [true|false] 是否以冒号为分隔符
 * @property {String} splitorColor 分割符号颜色
 * @event {Function} timeup 倒计时时间到触发事件
 * @example <wxy-countdown :day="1" :hour="1" :minute="12" :second="40"></wxy-countdown>
 *
 * 基于uni官方倒计时组件修改和增强
 * 修改说明:
 * 1、修改样式为自适应em,避免了调整的难度
 * 2、删除了多余的代码，如data中leftTime变量，还有startData和changeFlag方法中转换时间代码。
 * 增强说明：
 * 1、增加可设置用户时间功能，特别适合考试到计时，如还有1分钟时声音提醒和倒计时变颜色
 * 2、提供了暂停和继续功能，支持一开始就暂停，由用户来开始倒计时
 * @property {Number} splittime 秒 到这个时间会提醒，默认是0表示没有
 * @property {Boolean} pause = [true|false] true是暂停，false是继续，可控制开始就暂停
 * @event {Function} timesplit 用户定义时间到触发事件
 */
export default {
	name: 'UniCountdown',
	props: {
		showDay: {
			type: Boolean,
			default: true
		},
		showColon: {
			type: Boolean,
			default: true
		},
		backgroundColor: {
			type: String,
			default: '#FFFFFF'
		},
		borderColor: {
			type: String,
			default: '#000000'
		},
		color: {
			type: String,
			default: '#000000'
		},
		splitorColor: {
			type: String,
			default: '#000000'
		},
		day: {
			type: Number,
			default: 0
		},
		hour: {
			type: Number,
			default: 0
		},
		minute: {
			type: Number,
			default: 0
		},
		second: {
			type: Number,
			default: 0
		},
		// 设置用户想获取的时间
		splittime: {
			type: Number,
			default: 0
		},
		// 提供暂停和继续功能
		pause: {
			type: Boolean,
			default: false
		}
	},
	data() {
		return {
			timer: null,
			syncFlag: false,
			d: '00',
			h: '00',
			i: '00',
			s: '00',
			seconds: 0,
			over: false
		};
	},
	watch: {
		day(val) {
			this.changeFlag();
		},
		hour(val) {
			this.changeFlag();
		},
		minute(val) {
			this.changeFlag();
		},
		second(val) {
			this.changeFlag();
		},
		pause(val) {
			// 防止倒计时结束时又重新计时
			if(this.over) return false;
			if (val) {
				clearInterval(this.timer);
			} else {
				// 若是开始就暂停则按设置倒计时开始
				if (this.seconds == 0) this.startData(this.toSeconds(this.day, this.hour, this.minute, this.second));
				// 若是中音暂停则从中间开始
				else this.startData(this.seconds);
			}
		}
	},
	created: function(e) {
		// 若不是暂停则创建就倒计时
		if (!this.pause) this.startData(this.toSeconds(this.day, this.hour, this.minute, this.second));
		else {
			// 若是暂停则只显示时间
			this.seconds=this.toSeconds(this.day, this.hour, this.minute, this.second);
			this.countDown();
		}
	},
	beforeDestroy() {
		clearInterval(this.timer);
	},
	methods: {
		toSeconds(day, hours, minutes, seconds) {
			return day * 60 * 60 * 24 + hours * 60 * 60 + minutes * 60 + seconds;
		},
		// 倒计时结束时事件
		timeUp() {
			clearInterval(this.timer);
			// 设置状态为倒计时结束
			this.over = true;
			this.$emit('timeup');
		},
		// 用户设置时间到的事件
		timesplit(second) {
			this.$emit('timesplit', second);
		},
		countDown() {
			let seconds = this.seconds;
			let [day, hour, minute, second] = [0, 0, 0, 0];
			// 若是到了用户定义时间提醒用户
			if (this.splittime > 0 && seconds == this.splittime) this.timesplit(seconds);
			if (seconds > 0) {
				day = Math.floor(seconds / (60 * 60 * 24));
				hour = Math.floor(seconds / (60 * 60)) - day * 24;
				minute = Math.floor(seconds / 60) - day * 24 * 60 - hour * 60;
				second = Math.floor(seconds) - day * 24 * 60 * 60 - hour * 60 * 60 - minute * 60;
			} else {
				this.timeUp();
			}
			if (day < 10) {
				day = '0' + day;
			}
			if (hour < 10) {
				hour = '0' + hour;
			}
			if (minute < 10) {
				minute = '0' + minute;
			}
			if (second < 10) {
				second = '0' + second;
			}
			this.d = day;
			this.h = hour;
			this.i = minute;
			this.s = second;
		},
		startData(seconds) {
			this.seconds = seconds;
			if (this.seconds <= 0) {
				return;
			}
			this.countDown();
			this.timer = setInterval(() => {
				this.seconds--;
				if (this.seconds < 0) {
					this.timeUp();
					return;
				}
				this.countDown();
			}, 1000);
		},
		changeFlag() {
			if (!this.syncFlag) {
				this.startData(this.toSeconds(this.day, this.hour, this.minute, this.second));
				this.syncFlag = true;
			}
		}
	}
};
</script>
<style scoped>
.wxy-countdown {
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	flex-direction: row;
	justify-content: flex-start;
	padding: 0.1em 0;
}

.wxy-countdown__splitor {
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	justify-content: center;
	line-height: 1.4em;
	padding: 0.2em;
}

.wxy-countdown__number {
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	justify-content: center;
	align-items: center;
	width: 2em;
	height: 1.4em;
	line-height: 1.4em;
	margin: 0.2em;
	text-align: center;
}
</style>
