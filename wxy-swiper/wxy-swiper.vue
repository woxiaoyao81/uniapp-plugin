<template>
	<view
		class="wxy-swiper-wrap"
		:style="{
			borderRadius: `${borderRadius}rpx`
		}"
	>
		<swiper
			class="wxy-swiper-box"
			:current="elCurrent"
			@change="change"
			@animationfinish="animationfinish"
			:interval="interval"
			:circular="circular"
			:duration="duration"
			:autoplay="autoplay"
			:previous-margin="effect3d ? effect3dPreviousMargin + 'rpx' : '0'"
			:next-margin="effect3d ? effect3dPreviousMargin + 'rpx' : '0'"
			:style="{
				height: getHeight,
				backgroundColor: bgColor
			}"
		>
			<swiper-item class="wxy-swiper-item" v-for="(item, index) in list" :key="index">
				<view
					class="u-list-image-wrap"
					@tap.stop.prevent="listClick(index)"
					:class="[uCurrent != index ? 'u-list-scale' : '']"
					:style="{
						borderRadius: `${borderRadius}rpx`,
						transform: effect3d && uCurrent != index ? 'scaleY(0.9)' : 'scaleY(1)',
						margin: effect3d && uCurrent != index ? '0 20rpx' : 0
					}"
				>
					<!-- 增加自定义插槽 -->
					<block v-if="!haveSlot">
						<image class="wxy-swiper-image" :src="item[name[0]]||item[name] || item" :mode="imgMode"></image>
						<view
							v-if="title && item[name[1]]||item.title"
							class="wxy-swiper-title u-line-1"
							:style="[
								{
									'padding-bottom': titlePaddingBottom
								},
								titleStyle
							]"
						>
							{{ item[name[1]]||item.title }}
						</view>
					</block>
					<slot v-else :elem="item"></slot>
				</view>
			</swiper-item>
		</swiper>
		<view
			class="wxy-swiper-indicator"
			:style="{
				top: indicatorPos == 'topLeft' || indicatorPos == 'topCenter' || indicatorPos == 'topRight' ? '12rpx' : 'auto',
				bottom: indicatorPos == 'bottomLeft' || indicatorPos == 'bottomCenter' || indicatorPos == 'bottomRight' ? '12rpx' : 'auto',
				justifyContent: justifyContent,
				padding: `0 ${effect3d ? '74rpx' : '24rpx'}`
			}"
		>
			<block v-if="mode == 'rect'">
				<view class="u-indicator-item-rect" :class="{ 'u-indicator-item-rect-active': index == uCurrent }" v-for="(item, index) in list" :key="index"></view>
			</block>
			<block v-if="mode == 'dot'">
				<view class="u-indicator-item-dot" :class="{ 'u-indicator-item-dot-active': index == uCurrent }" v-for="(item, index) in list" :key="index"></view>
			</block>
			<block v-if="mode == 'round'">
				<view class="u-indicator-item-round" :class="{ 'u-indicator-item-round-active': index == uCurrent }" v-for="(item, index) in list" :key="index"></view>
			</block>
			<block v-if="mode == 'number'">
				<view class="u-indicator-item-number">{{ uCurrent + 1 }}/{{ list.length }}</view>
			</block>
		</view>
	</view>
</template>

<script>
/**
 * swiper 轮播图(基于uViewUI的swiper修改增强)
 * @description 该组件一般用于导航轮播，广告展示等场景,可开箱即用
 * @tutorial https://www.uviewui.com/components/swiper.html
 * @property {Array} list 轮播图数据，见官网"基本使用"说明
 * @property {Boolean} title 是否显示标题文字，需要配合list参数，见官网说明（默认false）
 * @property {String} mode 指示器模式，见官网说明（默认round）
 * @property {String Number} height 轮播图组件高度，单位rpx（默认250）
 * @property {String} indicator-pos 指示器的位置（默认bottomCenter）
 * @property {Boolean} effect3d 是否开启3D效果（默认false）
 * @property {Boolean} autoplay 是否自动播放（默认true）
 * @property {String Number} interval 自动轮播时间间隔，单位ms（默认2500）
 * @property {Boolean} circular 是否衔接播放，见官网说明（默认true）
 * @property {String} bg-color 背景颜色（默认#f3f4f6）
 * @property {String Number} border-radius 轮播图圆角值，单位rpx（默认8）
 * @property {Object} title-style 自定义标题样式
 * @property {String Number} effect3d-previous-margin mode = true模式的情况下，激活项与前后项之间的距离，单位rpx（默认50）
 * @property {String} img-mode 图片的裁剪模式，详见image组件裁剪模式（默认aspectFill）
 * @event {Function} click 点击轮播图时触发
 * @example <wxy-swiper :list="list" mode="dot" indicator-pos="bottomRight"></wxy-swiper>
 *
 * 修改说明:
 * 1、所有rpx单位已经换成em相对单位,适应用户定义的容器(2020-11-15)
 * 2、修改一些默认属性，减少配置。如取消默认自动播放(2020-11-15)
 * 3、轮播图高度调整为若是数字则默认单位是rpx,默认值是250,也可直接接受用户的px、%、auto、em、vw和vh等高度(2020-12-27)
 * 4、指定图片属性升级为支持数组指定图片和标题，格式如:name="['image','subject_name']"(2020-12-27)
 * 增强说明:
 * 1、增加自定义插槽功能，用户可以自定义展示功能，也可使用默认((2020-11-15))
 * @property {Boolean} haveSlot 是否使用插槽（默认false）V3引擎默认不支持APP的$slots判断。
 * V3引擎对Vue支持情况:https://uniapp.dcloud.io/use?id=%e5%ae%9e%e4%be%8b%e5%b1%9e%e6%80%a7
 * 测试支持平台:微信小程序、APP和H5
 * <u-swiper :list="list" :haveSlot="true" :effect3d="true" mode="none" @change='change'>
 * 	<template v-slot="{elem}">
 * 		<view class="tab ">
 * 			<image class="tab-img" :src="elem.imgUrl" mode=""></image>
 * 			<text class="tab-title f-bold f34 lb-6">{{elem.label}}</text>
 *			<view class="tab-tag">{{elem.munber}}个推荐地点</view>
 *		</view>
 *</template>
 *</u-swiper> * 
 */
export default {
	name: 'wxy-swiper',
	props: {
		// 轮播图的数据,格式如：[{image: 'xxxx', title: 'xxxx'}，{image: 'yyyy', title: 'yyyy'}]，其中title字段可选
		list: {
			type: Array,
			default() {
				return [];
			}
		},
		// 是否显示title标题
		title: {
			type: Boolean,
			default: false
		},
		// 用户自定义的指示器的样式
		indicator: {
			type: Object,
			default() {
				return {};
			}
		},
		// 圆角值
		borderRadius: {
			type: [Number, String],
			default: 8
		},
		// 隔多久自动切换
		interval: {
			type: [String, Number],
			default: 3000
		},
		// 指示器的模式，rect|dot|number|round
		mode: {
			type: String,
			default: 'round'
		},
		// list的高度，单位rpx
		height: {
			type: [Number, String],
			default: 250
		},
		// 指示器的位置，topLeft|topCenter|topRight|bottomLeft|bottomCenter|bottomRight
		indicatorPos: {
			type: String,
			default: 'bottomCenter'
		},
		// 是否开启缩放效果
		effect3d: {
			type: Boolean,
			default: false
		},
		// 3D模式的情况下，激活item与前后item之间的距离，单位rpx
		effect3dPreviousMargin: {
			type: [Number, String],
			default: 50
		},
		// 是否自动播放
		autoplay: {
			type: Boolean,
			default: false
		},
		// 自动轮播时间间隔，单位ms
		duration: {
			type: [Number, String],
			default: 500
		},
		// 是否衔接滑动，即到最后一张时接着滑动，是否自动切换到第一张
		circular: {
			type: Boolean,
			default: true
		},
		// 图片的裁剪模式
		imgMode: {
			type: String,
			default: 'aspectFill'
		},
		// 从list数组中读取的图片的属性
		// 修改说明：官方只支持图片指定，现在可以能完数组指定图片和标题，格式是:name="['image','subject_name']"
		name: {
			type: [String, Array],
			default: 'image'
		},
		// 背景颜色
		bgColor: {
			type: String,
			default: '#f3f4f6'
		},
		// 初始化时，默认显示第几项
		current: {
			type: [Number, String],
			default: 0
		},
		// 标题的样式，对象形式
		titleStyle: {
			type: Object,
			default() {
				return {};
			}
		},
		//  是否用插槽
		haveSlot: {
			type: Boolean,
			default: false
		}
	},
	watch: {
		// 如果外部的list发生变化，判断长度是否被修改，如果前后长度不一致，重置uCurrent值，避免溢出
		list(nVal, oVal) {
			if (nVal.length !== oVal.length) this.uCurrent = 0;
		},
		// 监听外部current的变化，实时修改内部依赖于此测uCurrent值，如果更新了current，而不是更新uCurrent，
		// 就会错乱，因为指示器是依赖于uCurrent的
		current(n) {
			this.uCurrent = n;
		}
	},
	data() {
		return {
			uCurrent: this.current // 当前活跃的swiper-item的index
		};
	},
	computed: {
		justifyContent() {
			if (this.indicatorPos == 'topLeft' || this.indicatorPos == 'bottomLeft') return 'flex-start';
			if (this.indicatorPos == 'topCenter' || this.indicatorPos == 'bottomCenter') return 'center';
			if (this.indicatorPos == 'topRight' || this.indicatorPos == 'bottomRight') return 'flex-end';
		},
		titlePaddingBottom() {
			let tmp = 0;
			if (this.mode == 'none') return '12rpx';
			if (['bottomLeft', 'bottomCenter', 'bottomRight'].indexOf(this.indicatorPos) >= 0 && this.mode == 'number') {
				tmp = '60rpx';
			} else if (['bottomLeft', 'bottomCenter', 'bottomRight'].indexOf(this.indicatorPos) >= 0 && this.mode != 'number') {
				tmp = '40rpx';
			} else {
				tmp = '12rpx';
			}
			return tmp;
		},
		// 因为uni的swiper组件的current参数只接受Number类型，这里做一个转换
		elCurrent() {
			return Number(this.current);
		},
		// 用户输入高度若是数字则默认单位是rpx，若是有其它则使用用户定义的单位
		getHeight() {
			if (/(px|%|rpx|em|vh|vw|auto)$/.test(this.height)) return this.height;
			else return this.height + 'rpx';
		}
	},
	methods: {
		listClick(index) {
			this.$emit('click', index);
		},
		change(e) {
			let current = e.detail.current;
			this.uCurrent = current;
			// 发出change事件，表示当前自动切换的index，从0开始
			this.$emit('change', current);
		},
		// 头条小程序不支持animationfinish事件，改由change事件
		// 暂不监听此事件，因为不再给swiper绑定uCurrent属性
		animationfinish(e) {
			// #ifndef MP-TOUTIAO
			// this.uCurrent = e.detail.current;
			// #endif
		}
	}
};
</script>

<style lang="scss" scoped>
.wxy-swiper-wrap {
	position: relative;
	overflow: hidden;
	transform: translateY(0);
}

.wxy-swiper-image {
	width: 100%;
	will-change: transform;
	height: 100%;
	/* #ifndef APP-NVUE */
	display: block;
	/* #endif */
	/* #ifdef H5 */
	pointer-events: none;
	/* #endif */
}

.wxy-swiper-indicator {
	padding: 0 1em;
	position: absolute;
	display: flex;
	width: 100%;
	z-index: 1;
}

.u-indicator-item-rect {
	width: 1em;
	height: 0.3em;
	margin: 0 0.25em;
	transition: all 0.5s;
	background-color: rgba(0, 0, 0, 0.3);
}

.u-indicator-item-rect-active {
	background-color: rgba(255, 255, 255, 0.8);
}

.u-indicator-item-dot {
	width: 0.5em;
	height: 0.5em;
	margin: 0 0.25em;
	border-radius: 0.8em;
	transition: all 0.5s;
	background-color: rgba(0, 0, 0, 0.3);
}

.u-indicator-item-dot-active {
	background-color: rgba(255, 255, 255, 0.8);
}

.u-indicator-item-round {
	width: 0.5em;
	height: 0.5em;
	margin: 0 0.25em;
	border-radius: 0.8em;
	transition: all 0.5s;
	background-color: rgba(0, 0, 0, 0.3);
}

.u-indicator-item-round-active {
	width: 1.5em;
	background-color: rgba(255, 255, 255, 0.8);
}

.u-indicator-item-number {
	padding: 0.25em 0.7em;
	line-height: 1;
	background-color: rgba(0, 0, 0, 0.3);
	border-radius: 4.5em;
	color: rgba(255, 255, 255, 0.8);
}

.u-list-scale {
	transform-origin: center center;
}

.u-list-image-wrap {
	width: 100%;
	height: 100%;
	flex: 1;
	transition: all 0.5s;
	overflow: hidden;
	box-sizing: content-box;
	position: relative;
}

.wxy-swiper-title {
	position: absolute;
	background-color: rgba(0, 0, 0, 0.3);
	bottom: 0;
	left: 0;
	width: 100%;
	padding: 0.5em 1em;
	color: rgba(255, 255, 255, 0.9);
}

.wxy-swiper-item {
	display: flex;
	overflow: hidden;
	align-items: center;
}
</style>
