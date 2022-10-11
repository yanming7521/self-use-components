<template>
	<view class="m-slider-container">
		<slider ref="slider" class="m-slider" v-bind="$attrs" @change="change" />
		<view class="range-text" v-if="showStartEnd">
			<text>{{ $attrs.min }}{{ unit }}</text>
			<text>{{ $attrs.max }}{{ unit }}</text>
		</view>
		<view class="step-text" v-if="showTick">
			<text v-for="(item, index) in stepList" :key="item" class="text" :class="{ active: slideValue === item }" :style="{ left: left(index) }">{{ item }}{{ unit }}</text>
		</view>
	</view>
</template>
<script>
export default {
	props: {
		unit: {
			type: String,
			default: ''
		},
		showStartEnd: {
			type: Boolean,
			default: false
		},
		showTick: {
			type: Boolean,
			default: false
		}
	},
	data() {
		return {
			slideValue: this.$attrs.value || 1
		};
	},
	computed: {
		stepList() {
			const { min, max, step } = this.$attrs;
			let arr = [];
			for (let i = min; i <= max; i = i + (step || 1)) {
				arr.push(i);
			}
			return arr;
		}
	},
	methods: {
		change(e) {
			this.slideValue = e.detail.value;
			this.$emit('change', this.slideValue);
		},
		left(index) {
			return index * (100 / (this.stepList.length - 1)) + '%';
		}
	}
};
</script>
<style lang="scss" scoped>
.m-slider-container {
	.m-slider {
		margin: 0;

		/deep/ {
			// 高度
			.uni-slider-handle-wrapper {
				height: 16rpx;
			}

			// 滑块圆环
			.uni-slider-thumb {
				box-shadow: 0 0 0 4rpx #ab0711;
				width: 20rpx !important;
				height: 20rpx !important;
				margin-left: -6rpx !important;
				margin-top: -10.5rpx !important;
			}

			// 滑动轨迹 track
			.uni-slider-track {
				background-image: linear-gradient(270deg, #cc1a11 0%, #ab0711 99%);
				border-radius: 5px;
			}

			.uni-slider-value {
				width: 2.5ch;
				color: #cc1a11;
				font-size: 34rpx;
				line-height: 34rpx;
				margin-left: 1em;
			}
		}
	}

	// 最大最小值
	.range-text {
		display: flex;
		align-items: center;
		justify-content: space-between;
		font-size: 24rpx;
	}

	// 步长
	.step-text {
		margin-top: 20rpx;
		height: 36rpx;
		position: relative;

		.text {
			position: absolute;
			top: 0;
			margin-left: -13px;
			text-align: center;
			width: 56rpx;
			font-size: 24rpx;
			color: #333;

			&.active {
				color: #4072ff;
			}
		}
	}
}
</style>
