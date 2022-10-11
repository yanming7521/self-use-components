<template>
	<view class="con">
		<swiper :style="'height:'+sHeight+'px'" class="content" :current="nowexam" @change='nowexamchange'>
			<swiper-item v-for="(item,index) in examList" :key='index'>
				<scroll-view scroll-y :style="'height:'+sHeight+'px'" style="width: 100%;">
					<view class="c_fir">
						<view class="cf_left">
							<text class="cl_text" v-text="item.type==0?'单选题':'多选题'"></text>
						</view>
						<view class="cf_right">
							<text class="cr_text1" v-text="index+1"></text>
							<text class="cr_text2" v-text="'/'+examTotal"></text>
						</view>
					</view>
					<view class="c_sec">
						<view class="q_title">
							<u-parse :content="item.title" :imageProp="imgprop" :noData="noData" />
						</view>
						<view class="q_choose">
							<!-- 单选题 -->
							<view class='qc_dx' v-if="item.type==0">
								<radio-group @change='answerChange'>
									<label class="pd_item" v-for="(it,index2) in item.answerList" :key='index2'>
										<view class="pdi_left">
											<radio :value="it.value" :checked="item.userAnswer==it.value?true:false" style="transform:scale(0.8)" color="#62B6FF"></radio>
										</view>
										<view class="pdi_right" v-text="it.label"></view>
									</label>
								</radio-group>
							</view>
							<!-- 多选题 -->
							<view class="qc_dx" v-if="item.type==1">
								<checkbox-group @change='answerChange'>
									<label class="pd_item" v-for="(it,index2) in item.answerList" :key='index2'>
										<view class="pdi_left">
											<checkbox :value="it.value" style="transform:scale(0.8)" :checked="item.userAnswer.indexOf(it.value)==-1?false:true"  color="#62B6FF"></checkbox>
										</view>
										<view class="pdi_right" v-text="it.label"></view>
									</label>
								</checkbox-group>
							</view>
							
						</view>
						<view class="q_btn">
							<view class="qb_next" v-if='index!=0' @click="nowchangeBybtn(-1)">上一题</view>
							<view class="qb_next" v-if='index!=examTotal-1' @click="nowchangeBybtn(1)">下一题</view>
							<view class="qb_next" v-if='index==examTotal-1' @click="gosubmit">交卷</view>
						</view>
					</view>
					<view class="c_thi" v-if="issubmit">
						<view class="ct_fir">
							<view class="ctf_item">正确答案： <text class="ci_t" v-text="item.answer"></text></view>
							<view class="ctf_item">你的答案： <text :class="item.answer==item.userAnswer?'ci_c':'ci_w'" v-text="item.userAnswer"></text></view>
						</view>
						<view class="ct_sec">
							<view class="cts_left">
								<i class='iconfont iconshuxian' style='font-size: 46upx;color: #62B6FF;'></i>
							</view>
							<view class="cts_right">解析</view>
						</view>
						<view class="ct_thi" v-text="item.example"></view>
					</view>			
				</scroll-view>
				
			</swiper-item>
		</swiper>
		<view class="answer" @click="showDTK">
			<i class='iconfont icondatiqia' style='font-size: 46upx;color: #fff;'></i>
		</view>
		<uni-popup ref="popup" type="center">
			<view class="dtk">
				<view class="d_title">
					<view class="ct_left">答题卡</view>
					<view class="ct_right">
						<uni-icons @click="closeDTK" type="closeempty" size="20" color="#333"></uni-icons>
					</view>
				</view>
				<scroll-view style="width: 100%;height: 71vh;" scroll-y>
					<view class="d_con">
						<view class="dc_item" v-for="(item,index) in examList" :key='index'>
							<view @click="nowexamchangeByDTK(index)" :class="item.userAnswer==''?(tempUserAnswer==''?'dci_num2':(nowexam==index?'dci_num':'dci_num2')):'dci_num'" v-text="index+1"></view>
						</view>
					</view>
				</scroll-view>
				<view class="d_btn" @click="gosubmit">提交试卷</view>
				<view class="d_explain">
					<view class="de_fir">
						<view class="df_item"></view>
						<view class="df_item"></view>
					</view>
				</view>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	import uParse from '@/components/pxing-exam/u-parse/u-parse.vue';
	import uniIcons from "@/components/pxing-exam/uni-icons/uni-icons.vue";
	import uniPopup from '@/components/pxing-exam/uni-popup/uni-popup.vue';
	import uniPopupMessage from '@/components/pxing-exam/uni-popup/uni-popup-message.vue';
	import uniPopupDialog from '@/components/pxing-exam/uni-popup/uni-popup-dialog.vue';
	export default {
		name:'pxingExam',
		components: {
			uParse,uniIcons,uniPopup,uniPopupMessage,uniPopupDialog
		},
		computed:{
			noData(){
				return "";
			},
			imgprop(){
				let imgprop = {
					lozyLoad:true,
					domain:""
				}
				return imgprop;
			}
		},
		props:{
			nownum:{ //当前题目num
				type:Number,
				default:0
			},
			examlist:{ //考试题目列表
				type:Array,
				default:[]
			},
			submitType:{
				type:Number,
				default:0
			}
		},
		watch:{
			nownum:{
				immediate: true,
				handler(n,o){
					this.nowexam = n;
				}
			},
			examlist:{
				immediate: true,
				handler(n,o){
					this.examList = n;
				}
			},
			submitType:{
				immediate: true,
				handler(n,o){
					this.sutype = n;
				}
			}
		},
		data() {
			return {
				sHeight: 0,
				sutype: this.$props.submitType,
				nowexam: this.$props.nownum,
				examList: this.$props.examlist,
				examTotal: 0,
				tempUserAnswer:'',
				examchangeByDTK:false,
				dtknum:0,
				issubmit:false,
				lastUserAnswer:[]
			};
		},
		mounted() {
			this.getData();
			this.getheight();
		},
		methods:{
			nowchangeBybtn(num){
				this.examchangeByDTK = true;
				this.dtknum = this.nowexam;
				this.nowexam = this.nowexam + num;
			},
			nowexamchangeByDTK(index){
				this.examchangeByDTK = true;
				this.dtknum = this.nowexam;
				this.nowexam = index;
				this.$refs.popup.close();
			},
			gosubmit(){
				if(this.issubmit==false){
					if(this.tempUserAnswer!=''){
						if(this.examList[this.nowexam].type==0){
							this.examList[this.nowexam].userAnswer = this.tempUserAnswer;
						}else if(this.examList[this.nowexam].type==1){
							var tempList = this.tempUserAnswer;
							for(var i=0;i<tempList.length;i++){
								for(var j=i+1;j<tempList.length;j++){
									if(tempList[i]>tempList[j]){
										var temp = tempList[i];
										tempList[i] = tempList[j];
										tempList[j] = temp;
									}
								}
							}
							var templast = '';
							for(var i=0;i<tempList.length;i++){
								templast = templast + tempList[i];
							}
							this.examList[this.nowexam].userAnswer = templast;
						}
					}
					if(this.sutype==0){
						this.$emit('getEveryAnswer',this.examList[this.nowexam].userAnswer);
					}else if(this.sutype==1){
						for(var i=0;i<this.examList.length;i++){
							this.lastUserAnswer[i]=this.examList[i].userAnswer;
						}
						this.$emit('getLastAnswer',this.lastUserAnswer);
					}
					this.issubmit = true;
					this.$refs.popup.close();
				}else{
					uni.showToast({
						title: '答案已提交，请勿重复提交！',
						icon: 'none'
					})
				}
			},
			closeDTK(){
				this.$refs.popup.close();
			},
			showDTK(){
				this.$refs.popup.open();
			},
			answerChange(e){
				if(this.examList[this.nowexam].type==0){
					this.tempUserAnswer = e.detail.value;
				}else{
					var tempList = e.detail.value;
					for(var i=0;i<tempList.length;i++){
						for(var j=i+1;j<tempList.length;j++){
							if(tempList[i]>tempList[j]){
								var temp = tempList[i];
								tempList[i] = tempList[j];
								tempList[j] = temp;
							}
						}
					}
					var templast = '';
					for(var i=0;i<tempList.length;i++){
						templast = templast + tempList[i];
					}
					this.tempUserAnswer = templast;
				}
			},
			nowexamchange(e){
				var tempindex = e.detail.current;
				if(this.examchangeByDTK){
					if(this.tempUserAnswer!=''){
						if(this.examList[this.dtknum].userAnswer==''){
							this.examList[this.dtknum].userAnswer = this.tempUserAnswer;
							if(this.sutype==0){
								this.$emit('getEveryAnswer',this.examList[this.dtknum].userAnswer);
							}
						}
						this.examchangeByDTK = false;
						this.tempUserAnswer = '';
					}
				}else{
					if(this.tempUserAnswer!=''){
						if(this.examList[this.nowexam].userAnswer==''){
							this.examList[this.nowexam].userAnswer = this.tempUserAnswer;
							if(this.sutype==0){
								this.$emit('getEveryAnswer',this.examList[this.nowexam].userAnswer);
							}
						}
						this.tempUserAnswer = '';
					}
				}
				this.nowexam = tempindex;
				
			},
			getData(){
				this.examTotal = this.examList.length;
			},
			async getheight(){
				var sh = 0;
				var that = this;
				var windowInfo = uni.getSystemInfo({
					success(res){
						that.sHeight = res.windowHeight;
					}
				});
			}
			
		}
	}
</script>

<style lang="scss" scoped>
	@import "@/components/pxing-exam/iconfont.css";
	.con{
		width: 100%;
		.content{
			width: 100%;
			.c_fir{
				width: 94%;
				padding: 0 3%;
				height: 80upx;
				background-color: #f6f6f6;
				display: flex;
				.cf_left{
					width: 70%;
					height: 80upx;
					line-height: 80upx;
					.cl_text{
						font-size: 28upx;
						color: #999;
					}
				}
				.cf_right{
					width: 30%;
					height: 80upx;
					line-height: 80upx;
					text-align: right;
					.cr_text1{
						font-size: 28upx;
						color: #62B6FF;
					}
					.cr_text2{
						font-size: 28upx;
						color: #999;
					}
				}
			}
			.c_sec{
				width: 94%;
				margin: 0 auto;
				.q_title{
					width: 100%;
					margin-top: 20upx;
					font-size: 28upx;
					color: #333;
				}
				.q_choose{
					width: 100%;
					.qc_dx{
						width: 100%;
						margin-top: 24upx;
						padding-bottom: 24upx;
						.pd_item{
							width: 100%;
							display: flex;
							.pdi_left{
								width: 10%;
								padding: 12upx 0;
							}
							.pdi_right{
								width: 90%;
								padding: 16upx 0;
								font-size: 28upx;
							}
						}
					}
				}
				.q_btn{
					width: 98%;
					display: flex;
					justify-content: flex-end;
					height: 60upx;
					margin-bottom: 24upx;
					.qb_next{
						opacity: 0.8;
						margin-right: 22upx;
						width: 130upx;
						height: 60upx;
						line-height: 60upx;
						background-color: #62B6FF;
						color: #fff;
						text-align: center;
						font-size: 26upx;
						border-radius: 120upx;
					}
				}
			}
			.c_thi{
				width: 100%;
				padding: 24upx 0;
				.ct_fir{
					width: 94%;
					padding: 0 3%;
					display: flex;
					height: 80upx;
					.ctf_item{
						width: fit-content;
						margin-right: 3%;
						height: 80upx;
						display: flex;
						align-items: center;
						justify-content: center;
						font-size: 26upx;
						.ci_t{
							color: #62B6FF;
						}
						.ci_c{
							color: #2c9678;
						}
						.ci_w{
							color: #ee2746;
						}
					}
				}
				.ct_sec{
					width: 94%;
					margin: 20upx auto 0;
					height: 80upx;
					display: flex;
					border-bottom: 1px solid #f6f6f6;
					.cts_left{
						width: 6%;
						height: 80upx;
						display: flex;
						align-items: center;
						justify-content: center;
						.cl_img{
							width: 50upx;
							height: 32upx;
						}
					}
					.cts_right{
						width: 80%;
						height: 80upx;
						font-size: 28upx;
						color: #333;
						font-weight: 700;
						line-height: 80upx;
					}
				}
				.ct_thi{
					width: 94%;
					margin: 0 3%;
					font-size: 26upx;
					color: #666;
					margin-top: 20upx;
					padding-bottom: 20upx;
				}
			}
		}
		.answer{
			border-radius: 50%;
			width: 80upx;
			height: 80upx;
			background-color: #62B6FF;
			opacity: 0.8;
			position: fixed;
			bottom: 80upx;
			right: 20upx;
			display: flex;
			align-items: center;
			justify-content: center;
			.a_img{
				width: 48upx;
				height: 48upx;
			}
		}
		.dtk{
			width: 86vw;
			height: 80vh;
			background-color: #fff;
			border-radius: 8upx;
			.d_title{
				width: 94%;
				padding: 0 3%;
				height: 7vh;
				display: flex;
				border-bottom: 1px solid #f6f6f6;
				.ct_left{
					width: 80%;
					height: 7vh;
					font-size: 30upx;
					color: #333;
					letter-spacing: 4upx;
					font-weight: 700;
					line-height: 7vh;
				}
				.ct_right{
					width: 20%;
					height: 7vh;
					display: flex;
					align-items: center;
					justify-content:flex-end;
				}
			}
			.d_con{
				width: 94%;
				padding-top: 12upx;
				margin: 0 auto;
				display: flex;
				flex-wrap: wrap;
				.dc_item{
					width: 20%;
					display: flex;
					align-items: center;
					justify-content: center;
					height: 100upx;
					.dci_num{
						width: 60upx;
						height: 60upx;
						background-color: #62B6FF;
						border-radius: 50%;
						text-align: center;
						line-height: 60upx;
						color: #fff;
						font-size: 24upx;
					}
					.dci_num2{
						width: 60upx;
						height: 60upx;
						background-color: #eee;
						border-radius: 50%;
						text-align: center;
						line-height: 60upx;
						color: #999;
						font-size: 24upx;
					}
				}
			}
			.d_btn{
				width: 100%;
				height: 6vh;
				background-color: #62B6FF;
				position: absolute;
				text-align: center;
				line-height: 6vh;
				color: #fff;
				font-size: 28upx;
				letter-spacing: 4upx;
				bottom: 0;
				border-radius: 0 0 8upx 8upx;
			}
		}
	}
</style>
