<template>
	<view class="HMfilterDropdown" :class="{'setDropdownBottom':maskVisibility}" :style="{'top':menuTop+'rpx'}"
		@touchmove.stop.prevent="discard" @tap.stop="discard" ref="xialaheight">
		<!-- 顶部菜单 -->
		<view class="nav2" :class="{ 'sticky-component': isSticky }">
			<block v-for="(item,index) in menu" :key="index">
				<view class="first-menu" :class="{'on':showPage==index}" @tap="togglePage(index)">
					<text class="name">{{item.name}}</text>
					<text class="iconfont triangle" :style="'transform:rotate('+triangleDeg[index]+'deg);'"></text>
				</view>
			</block>
		</view>
		<!-- 遮罩层 -->
		<view class="mask" :class="{'show':isShowMask,'hide':maskVisibility!=true}" @tap="hideMenu(false)"></view>
		<block v-for="(page,page_index) in subData" :key="page_index">
			<view class="sub-menu-class" :class="{'show':showPage==page_index,'hide':pageState[page_index]!=true}">
				<!-- 多级菜单 -->
				<block v-if="(page.type=='hierarchy'||page.type=='hierarchy-column')&& page.submenu.length>0">
					<!-- 第一级菜单 -->
					<scroll-view class="sub-menu-list"
						:class="{'first':activeMenuArr[page_index].length>1,'alone':activeMenuArr[page_index].length<=1}"
						:scroll-y="true" :scroll-into-view="'first_id'+firstScrollInto">
						<block v-for="(sub,index) in page.submenu" :key="sub.value">
							<view class="sub-menu" :id="'first_id'+index"
								:class="{'on':activeMenuArr[page_index][0]==index}"
								@tap="selectHierarchyMenu(page_index,index,null,null)">
								<view class="menu-name">
									<text>{{sub.name}}</text>
									<text class="iconfont selected"></text>
								</view>
							</view>
						</block>
					</scroll-view>
					<block v-if="page.type=='hierarchy'">
						<block v-for="(sub,index) in page.submenu" :key="sub.value">
							<!-- 第二级菜单 -->
							<scroll-view class="sub-menu-list not-first" :scroll-y="true"
								v-if="activeMenuArr[page_index][0]==index&&sub.submenu.length>0"
								:scroll-into-view="'second_id'+secondScrollInto">
								<block v-for="(sub_second,second_index) in sub.submenu" :key="sub_second.value">
									<view class="sub-menu" :id="'second_id'+second_index"
										:class="{'on':activeMenuArr[page_index][1]==second_index}">
										<view class="menu-name"
											@tap="selectHierarchyMenu(page_index,activeMenuArr[page_index][0],second_index,null)">
											<text>{{sub_second.name}}</text>
											<text class="iconfont selected"></text>
										</view>
										<!-- 第三级菜单 -->
										<view class="more-sub-menu"
											v-if="sub_second.submenu&&sub.submenu.length>0&&sub_second.submenu.length>0">
											<block v-for="(sub2,sub2_index) in sub_second.submenu" :key="sub2.value">
												<text v-if="sub_second.showAllSub || (sub2_index<8)"
													:class="{'on':activeMenuArr[page_index][1]==second_index&&activeMenuArr[page_index][2]==sub2_index}"
													@tap.stop="selectHierarchyMenu(page_index,activeMenuArr[page_index][0],second_index,sub2_index)">{{sub2.name}}</text>
												<text
													v-if="sub_second.showAllSub!=true && sub2_index==8 && sub_second.submenu.length>9"
													@tap.stop="showMoreSub(second_index)">更多<text
														class="iconfont triangle"></text></text>
											</block>
										</view>
									</view>
								</block>
							</scroll-view>
						</block>
					</block>

					<!-- 二三级菜单都是行形态 -->
					<block v-else-if="page.type=='hierarchy-column'">
						<!-- 第二级菜单 -->
						<block v-for="(sub,index) in page.submenu" :key="index">
							<scroll-view class="sub-menu-list not-first" :scroll-y="true"
								v-if="activeMenuArr[page_index][0]==index&&sub.submenu.length>0"
								:scroll-into-view="'second_id'+secondScrollInto">
								<block v-for="(sub_second,second_index) in sub.submenu" :key="second_index">
									<view class="sub-menu" :id="'second_id'+second_index"
										:class="{'on':activeMenuArr[page_index][1]==second_index}">
										<view class="menu-name"
											@tap="selectHierarchyMenu(page_index,activeMenuArr[page_index][0],second_index,null)">
											<text>{{sub_second.name}}</text>
										</view>
									</view>
								</block>
							</scroll-view>
						</block>
						<!-- 第三级菜单 -->
						<block v-for="(sub,index) in page.submenu">
							<block v-for="(sub_second,second_index) in sub.submenu">
								<scroll-view class="sub-menu-list not-first third" :scroll-y="true"
									v-if="activeMenuArr[page_index][0]==index&&activeMenuArr[page_index][1]==second_index&&sub_second.submenu.length>0"
									:scroll-into-view="'third_id'+thirdScrollInto">
									<block v-for="(sub2,sub2_index) in sub_second.submenu" :key="sub2_index">
										<view class="sub-menu" :id="'third_id'+sub2_index"
											:class="{'on':activeMenuArr[page_index][2]==sub2_index}">
											<view class="menu-name"
												@tap="selectHierarchyMenu(page_index,activeMenuArr[page_index][0],second_index,sub2_index)">
												<text>{{sub2.name}}</text>
											</view>
										</view>
									</block>
								</scroll-view>
							</block>
						</block>
					</block>


				</block>


				<!-- 多选筛选 -->
				<block v-if="page.type=='filter'">
					<view class="filter">
						<scroll-view class="menu-box" :scroll-y="true">
							<view class="box" v-for="(box,box_index) in page.submenu" :key="box_index">
								<view class="title">{{box.name}}</view>
								<view class="labels">
									<view v-for="(label,label_index) in box.submenu" :key="label_index"
										@tap="selectFilterLabel(page_index,box_index,label_index)"
										:class="{'on':label.selected}">{{label.name}}</view>
								</view>
							</view>

							<view class="btn-box">
								<view class="reset" @tap="resetFilterData(page_index)">重置</view>
								<view class="submit" @tap="setFilterData(page_index)">确定</view>
							</view>
						</scroll-view>
					</view>
				</block>

				<!-- 单选筛选 -->
				<block v-if="page.type=='radio'">
					<view class="filter2">
						<scroll-view class="menu-box2" :scroll-y="true">
							<view class="box2" v-for="(box,box_index) in page.submenu" :key="box_index">

								<view class="title2">{{box.name}}</view>
								<view class="labels2" style="border-bottom: solid 1rpx #eee;padding-bottom: 20rpx;">
									<view style="" v-for="(label,label_index) in box.submenu" :key="label_index"
										@tap="selectRadioLabel(page_index,box_index,label_index)">

										<!-- <image v-if="label.selected" src="../../../../static/xuanzhong.png" style="width: 40rpx;height: 40rpx;">
										</image>
										<image v-else src="../../../../static/weixuanzhong.png">
										</image> -->
										<!-- <view>
											<image src="../../../../static/xuanzhong.png" style="width: 40rpx;height: 40rpx;vertical-align: middle;"></image>
										</view> -->
										<view>
											<image
												:src="label.selected ? 'https://bn.duidian.cc/assets/addons/qingdongams/static/xuanzhong.png' : 'https://bn.duidian.cc/assets/addons/qingdongams/static/weixuanzhong.png'"
												style="width: 35rpx; height: 35rpx; vertical-align: middle;"></image>
										</view>
										<view class="lab" :class="{'on':label.selected}">
											{{label.name}}
										</view>
									</view>
								</view>
							</view>
						</scroll-view>
						<view class="btn-box2">
							<view class="reset" @tap="resetFilterData(page_index)">重置</view>
							<view class="submit" @tap="setFilterData(page_index)">确定</view>
						</view>

					</view>
				</block>
			</view>
		</block>
	</view>
</template>
<script>
	export default {
		name: 'HM-filterDropdown',
		data() {
			return {
				isSticky: false,
				menuData: [],
				subData: [], //菜单数据
				menu: [], //顶部菜单数据
				showPage: -1, //菜单页面显示/隐藏动画控制
				pageState: [], //页面的状态
				activeMenuArr: [], //UI状态
				shadowActiveMenuArr: [], //记录选中
				defaultActive: [],
				triangleDeg: [], //小三角形的翻转动画控制
				isShowMask: false, //遮罩层显示/隐藏动画控制
				maskVisibility: false, //遮罩层显示/隐藏状态

				//滚动区域定位
				firstScrollInto: 0,
				secondScrollInto: 0,
				thirdScrollInto: 0,
				componentTop: 0, //组件top
				isReadNewSelect: false,
			}
		},
		props: {
			menuTop: {
				value: Number,
				default: false
			},
			filterData: {
				value: Array,
				default: []
			},
			defaultSelected: {
			    type: Object, // 或 Array，根据实际情况选择类型
			    default: function () {
			      return {}; // 或 []，根据实际情况返回默认值
			    }
			},
			updateMenuName: {
				value: Boolean,
				default: true
			},
			dataFormat: {
				value: String,
				default: 'Array'
			}
		},
		watch: {
			filterData: {
				handler(newVal) {
					console.log('watch filterData');
					this.menuData = JSON.parse(JSON.stringify(newVal));
					this.initMenu(); //filterData重新赋值初始化菜单
				},
				immediate: true,
				deep: true
			},
			defaultSelected(newVal) {

				if (newVal.length == 0) {
					return;
				}
				this.defaultActive = JSON.parse(JSON.stringify(newVal));
				this.activeMenuArr = JSON.parse(JSON.stringify(newVal));
				this.shadowActiveMenuArr = JSON.parse(JSON.stringify(newVal));
				if (this.updateMenuName) {
					this.setMenuName();
				}
			}
		},

		methods: {
			initMenu() {
				let tmpMenuActiveArr = [];
				let tmpMenu = [];
				for (let i = 0; i < this.menuData.length; i++) {
					let tmpitem = this.menuData[i];
					tmpMenu.push({
						//如果没有设置name，则取第一个菜单作为menu.name,filter类型则将"筛选"作为menu.name
						name: tmpitem.name || (tmpitem.type == "filter" ? "筛选" : tmpitem.submenu[0].name),
						type: tmpitem.type
					});
					//初始化选中项数组-ui状态
					tmpMenuActiveArr.push(this.processActive(tmpitem));
					//初始化角度数组
					this.triangleDeg.push(0);
					//初始化控制显示状态数组
					this.pageState.push(false);
					//递归处理子菜单数据
					tmpitem = this.processSubMenu(tmpitem);
					this.menuData[i] = tmpitem;
				}
				this.menu = tmpMenu;
				//初始化选中项数组
				tmpMenuActiveArr = this.defaultActive.length > 0 ? this.defaultActive : this.activeMenuArr.length > 0 ?
					this.activeMenuArr : tmpMenuActiveArr;
				this.defaultActive = [];
				this.activeMenuArr.splice(0, this.activeMenuArr.length, ...JSON.parse(JSON.stringify(tmpMenuActiveArr)));
				// this.activeMenuArr = JSON.parse(JSON.stringify(tmpMenuActiveArr));
				this.shadowActiveMenuArr = JSON.parse(JSON.stringify(tmpMenuActiveArr));
				//加载菜单数据
				this.subData = this.menuData;
				//设定顶部菜单名字
				if (this.updateMenuName) {
					this.setMenuName();
				}
			},
			setMenuName() {
				for (var i = 0; i < this.activeMenuArr.length; i++) {
					let row = this.activeMenuArr[i];
					if (this.subData[i].type == 'hierarchy' || this.subData[i].type == 'hierarchy-column') {
						if (typeof(row[0]) == 'number') {
							let tmpsub = this.subData[i].submenu[row[0]];
							if (row.length > 1) {
								tmpsub = tmpsub.submenu[row[1]] || tmpsub;
								if (row.length > 2) {
									tmpsub = tmpsub.submenu[row[2]] || tmpsub;
								}
							}
							this.menu[i].name = tmpsub.name;
						} else {
							this.menu[i].name = this.subData[i].name;
						}
					}
				}
			},
			//展开更多
			showMoreSub(index) {
				this.subData[this.showPage].submenu[this.activeMenuArr[this.showPage][0]].submenu[index].showAllSub = true;
				this.$forceUpdate();
			},
			//选中
			selectHierarchyMenu(page_index, level1_index, level2_index, level3_index) {
				//读取记录
				if (level2_index == null && level3_index == null && this.shadowActiveMenuArr[page_index].length > 0 && this
					.shadowActiveMenuArr[page_index][0] == level1_index) {
					this.activeMenuArr.splice(page_index, 1, JSON.parse(JSON.stringify(this.shadowActiveMenuArr[
						page_index])));
				}
				/////////
				let tmpArr = new Array(this.activeMenuArr[page_index].length).fill(null);
				this.activeMenuArr[page_index].splice(0, this.activeMenuArr[page_index].length, ...tmpArr);
				////////
				this.activeMenuArr[page_index].splice(0, 1, level1_index);
				let tmpMemu = this.subData[page_index].submenu[level1_index];
				if (tmpMemu.submenu.length == 0) {
					this.selectedMemu(page_index, level1_index, level2_index, level3_index);
				} else if (level2_index != null) {
					this.activeMenuArr[page_index].splice(1, 1, level2_index);
					tmpMemu = tmpMemu.submenu[level2_index];
					if (tmpMemu.submenu.length == 0 || (this.menu[page_index].type == 'hierarchy' && level3_index ==
							null)) {
						this.selectedMemu(page_index, level1_index, level2_index, level3_index);
					} else if (level3_index != null) {
						this.activeMenuArr[page_index].splice(2, 1, level3_index);
						tmpMemu = tmpMemu.submenu[level3_index];
						this.selectedMemu(page_index, level1_index, level2_index, level3_index);
					}
				}
			},
			selectedMemu(page_index, level1_index, level2_index, level3_index) {
				let sub = this.subData[page_index].submenu[level1_index].submenu[level2_index];
				if (this.updateMenuName) {
					this.menu[page_index].name = (level3_index != null && sub.submenu[level3_index].name) || (
						level2_index != null && sub.name) || this.subData[page_index].submenu[level1_index].name;
				}
				this.shadowActiveMenuArr[page_index] = JSON.parse(JSON.stringify(this.activeMenuArr[page_index]));
				this.hideMenu(true);
			},
			//写入结果，筛选
			setFilterData(page_index) {
				this.shadowActiveMenuArr[page_index] = JSON.parse(JSON.stringify(this.activeMenuArr[page_index]));
				this.hideMenu(true);
			},
			//重置结果和ui，筛选
			resetFilterData(page_index) {
				let tmpArr = [];
				let level = this.shadowActiveMenuArr[page_index].length;
				while (level > 0) {
					tmpArr.push([]);
					let box = this.subData[page_index].submenu[level - 1].submenu;
					for (let i = 0; i < box.length; i++) {
						this.subData[page_index].submenu[level - 1].submenu[i].selected = false;
					}
					level--;
				}
				this.activeMenuArr[page_index] = JSON.parse(JSON.stringify(tmpArr));
				this.$forceUpdate();
			},
			//选中筛选类label-UI状态
			selectFilterLabel(page_index, box_index, label_index) {
				let find_index = this.activeMenuArr[page_index][box_index].indexOf(label_index);
				if (find_index > -1) {
					this.activeMenuArr[page_index][box_index].splice(find_index, 1);
					this.subData[page_index].submenu[box_index].submenu[label_index].selected = false;
				} else {
					this.activeMenuArr[page_index][box_index].push(label_index);
					this.subData[page_index].submenu[box_index].submenu[label_index].selected = true;
				}
				this.$forceUpdate();
			},
			//选中单选类label-UI状态
			selectRadioLabel(page_index, box_index, label_index) {

				let activeIndex = this.activeMenuArr[page_index][box_index][0];
				if (activeIndex == label_index) {
					this.subData[page_index].submenu[box_index].submenu[activeIndex].selected = false;
					this.activeMenuArr[page_index][box_index][0] = null;
				} else {
					if (activeIndex != null && activeIndex < this.subData[page_index].submenu[box_index].submenu.length) {
						this.subData[page_index].submenu[box_index].submenu[activeIndex].selected = false;
					}

					this.subData[page_index].submenu[box_index].submenu[label_index].selected = true;
					this.activeMenuArr[page_index][box_index][0] = label_index;
				}
				this.$forceUpdate();
			},
			//菜单开关
			togglePage(index) {
				if (this.isToggleing) {
					return;
				}
				this.isToggleing = true;
				if (index == this.showPage) {
					this.hideMenu();
					this.isSticky = false;
				} else {
					this.showMenu(index);
					this.isSticky = true;
				}
				setTimeout(() => {
					this.isToggleing = false;
				}, 150)
			},
			//hide菜单
			hideMenu(isTriggerConfirm) {
				this.hideMenuLayer(true);
				this.hideMaskLayer();
				this.showPage = -1;
				if (isTriggerConfirm) {
					this.confirm()
				}
				this.isSticky = false;
			},
			showMenu(index) {
				if (this.showPage > -1) {
					this.hideMenuLayer(false);
				}
				this.showMenuLayer(index);
				this.showMaskLayer();

			},
			//hide遮罩层
			hideMaskLayer() {
				this.isShowMask = false;
				setTimeout(() => {
					this.maskVisibility = false;
				}, 200);
			},
			//show遮罩层
			showMaskLayer() {
				this.maskVisibility = true;
				this.$nextTick(() => {
					setTimeout(() => {
						this.isShowMask = true;
					}, 0)
				})
			},
			//hide菜单页
			hideMenuLayer(isAnimation) {
				this.triangleDeg[this.showPage] = 0;
				let tmpIndex = this.showPage;
				if (isAnimation) {
					setTimeout(() => {
						this.pageState.splice(tmpIndex, 1, false);
					}, 200);
				} else {
					this.pageState.splice(tmpIndex, 1, false)
				}
				this.firstScrollInto = null;
				this.secondScrollInto = null;
			},
			//show菜单页
			showMenuLayer(index) {
				this.processPage(index);
				this.pageState.splice(index, 1, true);
				this.$nextTick(() => {
					setTimeout(() => {
						this.showPage = index;
					}, 0)
				})
				this.triangleDeg[index] = 180;
			},
			confirm() {
				let index = JSON.parse(JSON.stringify(this.shadowActiveMenuArr));
				let value = JSON.parse(JSON.stringify(this.shadowActiveMenuArr));

				//对结果做一下处理
				index.forEach((item, i) => {
					if (typeof(item[0]) == 'object') {
						//针对筛选结果过一个排序
						item.forEach((s, j) => {
							if (s != null) {
								s.sort((val1, val2) => {
									return val1 - val2;
								});
								item[j] = s;
								s.forEach((v, k) => {
									value[i][j][k] = (v == null || v >= this.subData[i].submenu[j]
											.submenu.length) ? null : this.subData[i].submenu[j]
										.submenu[v].value;
									if (this.subData[i].type == 'radio' && value[i][j][k] ==
										null) {
										value[i][j] = [];
										index[i][j] = [];
									}
								});
							}
						});
					} else {
						let submenu = this.subData[i].submenu[item[0]];
						value[i][0] = submenu.value;
						if (value[i].length >= 2 && item[1] != null) {
							if (submenu.submenu.length > 0) {
								submenu = submenu.submenu[item[1]];
								value[i][1] = submenu.hasOwnProperty('value') ? submenu.value : null;
							} else {
								value[i][1] = null
							}
							if (value[i].length >= 3 && item[2] != null) {
								if (submenu.submenu.length > 0) {
									submenu = submenu.submenu[item[2]];
									value[i][2] = submenu.hasOwnProperty('value') ? submenu.value : null;
								} else {
									value[i][2] = null;
								}
							}
						}
					}
					index[i] = item;

				});
				// 输出
				this.$emit('confirm', {
					index: index,
					value: value
				});
			},

			reloadActiveMenuArr() {
				for (let i = 0; i < this.menuData.length; i++) {
					let tmpitem = this.menuData[i];
					let tmpArr = this.processActive(tmpitem);
					tmpitem = this.processSubMenu(tmpitem);
					if (this.activeMenuArr[i].length != tmpArr.length) {
						this.menuData[i] = tmpitem;
						this.activeMenuArr.splice(i, 1, JSON.parse(JSON.stringify(tmpArr)));
						this.shadowActiveMenuArr.splice(i, 1, JSON.parse(JSON.stringify(tmpArr)));
					}
				}
				this.subData = this.menuData;
				this.$forceUpdate();
			},
			processPage(index) {
				//check UI控制数组，结果数组,防止传入数据层级和UI控制数组不同步
				this.reloadActiveMenuArr();
				//重置UI控制数组
				this.activeMenuArr.splice(index, 1, JSON.parse(JSON.stringify(this.shadowActiveMenuArr[index])));
				if (this.menu[index].type == 'filter') {
					//重载筛选页选中状态
					let level = this.shadowActiveMenuArr[index].length;
					for (let i = 0; i < level; i++) {
						let box = this.subData[index].submenu[i].submenu;
						for (let j = 0; j < box.length; j++) {
							if (this.shadowActiveMenuArr[index][i].indexOf(j) > -1) {
								this.subData[index].submenu[i].submenu[j].selected = true;
							} else {
								this.subData[index].submenu[i].submenu[j].selected = false;
							}
						}
					}
				} else if (this.menu[index].type == 'hierarchy') {
					this.$nextTick(() => {
						setTimeout(() => {
							//滚动到选中项
							this.firstScrollInto = parseInt(this.activeMenuArr[index][0]);
							this.secondScrollInto = parseInt(this.activeMenuArr[index][1]);
						}, 0);
					})
				} else if (this.menu[index].type == 'hierarchy-column') {
					this.$nextTick(() => {
						setTimeout(() => {
							//滚动到选中项
							this.firstScrollInto = parseInt(this.activeMenuArr[index][0]);
							this.secondScrollInto = parseInt(this.activeMenuArr[index][1]);
							this.thirdScrollInto = parseInt(this.activeMenuArr[index][2]);
						}, 0);
					})
				} else if (this.menu[index].type == 'radio') {
					//重载筛选页选中状态
					let level = this.shadowActiveMenuArr[index].length;
					for (let i = 0; i < level; i++) {
						let box = this.subData[index].submenu[i].submenu;
						for (let j = 0; j < box.length; j++) {
							if (this.shadowActiveMenuArr[index][i].indexOf(j) > -1) {
								this.subData[index].submenu[i].submenu[j].selected = true;
							} else {
								this.subData[index].submenu[i].submenu[j].selected = false;
							}
						}
					}
				}
			},
			processActive(tmpitem) {
				let tmpArr = []
				if ((tmpitem.type == 'hierarchy' || tmpitem.type == 'hierarchy-column') && tmpitem.hasOwnProperty(
						'submenu') && tmpitem.submenu.length > 0) {
					let level = this.getMaxFloor(tmpitem.submenu);
					while (level > 0) {
						tmpArr.push(null);
						level--;
					}
				} else if (tmpitem.type == 'filter') {
					let level = tmpitem.submenu.length;
					while (level > 0) {
						tmpArr.push([]);
						level--;
					}
				} else if (tmpitem.type == 'radio') {
					let level = tmpitem.submenu.length;
					while (level > 0) {
						tmpArr.push([]);
						level--;
					}
				}
				return tmpArr;
			},
			processSubMenu(menu) {
				if (menu.hasOwnProperty('submenu') && menu.submenu.length > 0) {
					for (let i = 0; i < menu.submenu.length; i++) {
						menu.submenu[i] = this.processSubMenu(menu.submenu[i]);
					}
				} else {
					menu.submenu = [];
				}
				return menu;
			},
			//计算菜单层级
			getMaxFloor(treeData) {
				let floor = 0
				let max = 0

				function each(data, floor) {
					data.forEach(e => {
						max = floor > max ? floor : max;
						if (e.hasOwnProperty('submenu') && e.submenu.length > 0) {
							each(e.submenu, floor + 1)
						}
					})
				}
				each(treeData, 1)
				return max;
			},
			discard() {

			}
		}
	}
</script>

<style>
	.sticky-component {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		/* 其他粘性布局的样式 */
	}
</style>
<style lang="scss">
	.HMfilterDropdown {
		// flex-shrink: 0;
		width: 100%;

		// position: fixed;
		// position: sticky;
		z-index: 9999;
		flex-wrap: nowrap;
		display: flex;
		flex-direction: row;
		// top: var(--window-top);
		// left:0;
		// top:100px;
		// overflow-y: hidden;

		&.setDropdownBottom {
			// height: 345px;
			bottom: 0;
		}

		view {
			display: flex;
			flex-wrap: nowrap;
		}
	}

	.region {
		flex: 1;
		height: 44px;
	}

	.nav2 {
		width: 100%;
		height: 44px;
		border-bottom: solid 1rpx #eee;
		z-index: 999;
		background-color: #ffffff;
		flex-direction: row;

		.first-menu {
			width: 100%;
			font-size: 28rpx;
			color: #000000;
			flex-direction: row;
			align-items: center;
			justify-content: center;
			transition: color .2s linear;

			&.on {
				color: #4c7fea;

				.iconfont {
					color: #4c7fea;
				}
			}

			.name {
				height: 20px;
				text-align: center;
				text-overflow: clip;
				// overflow: hidden;
			}

			.iconfont {
				width: 40rpx;
				height: 40rpx;
				font-size: 40rpx;
				align-items: center;
				justify-content: center;
				// transition: transform .2s linear, color .2s linear;
			}
		}
	}

	.sub-menu-class {
		width: 100%;
		position: absolute;
		left: 0;
		// transform: translate3d(0, - 100%, 0);
		// height: 66vh;
		background-color: #ffffff;
		z-index: 9999;
		box-shadow: 0 5px 5px rgba(0, 0, 0, .1);
		// overflow: hidden;
		flex-direction: row;
		transition: transform .15s linear;

		&.hide {
			display: none;
		}

		&.show {
			position: fixed;
			top: 0;
			left: 0;
			right: 0;
			transform: translate3d(0, calc(44px + 1rpx), 0);
		}
	}

	.sub-menu-list {
		width: 100%;
		// margin: 0 20rpx 0 20rpx;
		height: 94vh;
		flex-direction: column;

		.sub-menu {
			min-height: 44px;
			font-size: 28rpx;
			flex-direction: column;
			padding-right: 15px;

			>.menu-name {
				height: 44px;
				flex-direction: row;
				align-items: center;
				justify-content: space-between;

				>.iconfont {
					display: none;
					font-size: 40rpx;
					color: #4c7fea;
				}
			}
		}

		&.first {
			flex-shrink: 0;
			width: 284rpx;
			background-color: #ffffff;

			.sub-menu {
				padding-left: 15px;

				&.on {
					background-color: #fff;
				}
			}

			box-shadow: 5rpx 0 5rpx rgba($color: #000000, $alpha: 0.1);
		}

		&.alone {

			// max-height: 340px;
			// min-height: 170px;

			.sub-menu {
				min-height: calc(44px - 1rpx);
				margin-left: 15px;
				border-bottom: solid 1rpx #e5e5e5;

				&.on {
					color: #4c7fea;

					>.menu-name {
						>.iconfont {
							display: block;
						}
					}
				}
			}
		}

		&.third {
			// box-shadow: 5rpx 0 20rpx rgba($color: #000000, $alpha: 0.2) inset;
		}

		&.not-first {
			box-shadow: 5rpx 0 5rpx rgba($color: #000000, $alpha: 0.1);

			.sub-menu {
				min-height: calc(44px - 1rpx);
				margin-left: 15px;
				border-bottom: solid 1rpx #e5e5e5;

				>.menu-name {
					height: calc(44px - 1rpx);

					>.iconfont {
						display: none;
						font-size: 36rpx;
						color: #4f7de9;
					}
				}

				&.on {
					color: #4f7de9;

					>.menu-name {
						>.iconfont {
							display: block;
						}
					}
				}

				.more-sub-menu {
					flex-direction: row;
					flex-wrap: wrap;
					padding-bottom: 9px;

					>text {
						height: 30px;
						border-radius: 3px;
						background-color: #f5f5f5;
						color: #000000;
						margin-bottom: 6px;
						margin-right: 6px;
						text-align: center;
						line-height: 30px;
						border: solid #f5f5f5 1rpx;
						flex: 0 0 calc(33.33% - 6px);
						// overflow: hidden;
						font-size: 28rpx;

						&:nth-child(3n) {
							margin-right: 0;
						}

						&.on {
							border-color: #f6c8ac;
							color: #4c7fea;
						}

						.iconfont {
							color: #4c7fea;
						}
					}
				}
			}
		}
	}

	.filter {
		width: 100%;
		height: 68vh;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		align-items: center;

		.menu-box {
			margin: 0 20rpx;
			width: auto;
			height: auto;
			flex-shrink: 1;

			.box {
				width: 100%;
				padding-top: 6px;
				height: auto;
				flex-direction: column;

				.title {
					width: 100%;
					font-size: 28rpx;
					color: #55557f;
					margin-bottom: 30rpx;
				}

				.labels {
					flex-direction: row;
					flex-wrap: wrap;
					margin-bottom: 20rpx;

					.on {
						// border-color: #ec652b;
						background-color: #4c7fea;
						color: #fff;
					}

					>view {
						padding: 6rpx 18rpx;
						// box-sizing: border-box;
						width: auto;
						// width: calc((698rpx - 75rpx * 2) / 4);
						height: 40rpx;
						// border: solid 1rpx #adadad;
						background-color: #f9fafc;
						border-radius: 15rpx;
						margin-right: 25rpx;
						margin-top: 8px;
						font-size: 28rpx;
						flex-direction: row;
						justify-content: center;
						align-items: center;

						&:nth-child(4n) {}
					}
				}
			}
		}

		.btn-box {
			position: fixed;
			left: 0;
			right: 0;
			margin: 20rpx 20rpx 0 20rpx;
			flex-shrink: 0;
			width: auto;
			height: auto;
			// z-index: 9999;
			display: flex;
			// margin: 10rpx 0;
			flex-direction: row !important;
			align-items: center;
			justify-content: space-between;

			>view {
				width: 320rpx;
				height: 40px;
				border-radius: 20rpx;
				border: solid 1rpx #4c7fea;
				align-items: center;
				justify-content: center;
				// margin-bottom: 35rpx
			}

			.reset {
				margin-right: 10rpx;
				color: #4c7fea;
			}

			.submit {
				margin-left: 10rpx;
				color: #fff;
				background-color: #4c7fea;
			}
		}
	}

	.filter2 {
		width: 100%;
		height: 40vh;
		display: flex;
		flex-direction: column;

		.menu-box2 {
			margin: 0 20rpx;
			width: auto;
			height: auto;
			flex-shrink: 1;

			.box2 {
				width: 100%;
				// padding-top: 6px;
				height: auto;
				flex-direction: column;

				.title2 {
					width: 100%;
					font-size: 28rpx;
					color: #55557f;
					margin-bottom: 30rpx;
				}

				.labels2 {
					flex-direction: row;
					flex-wrap: wrap;
					// margin-bottom: 20rpx;

					view {
						// 所有 <view> 元素的样式
						display: flex;
						flex-direction: row;

						.lab {
							padding: 6rpx 18rpx;
							box-sizing: border-box;
							width: auto;
							// width: calc((698rpx - 75rpx * 2) / 4);
							height: 30rpx;
							// border: solid 1rpx #adadad;
							// background-color: #f9fafc;
							// border-radius: 15rpx;
							// margin-right: 25rpx;
							// margin-top: 8px;
							font-size: 28rpx;
							// flex-direction: row;
							justify-content: center;
							align-items: center;
						}

						.on {
							// border-color: #ec652b;
							// background-color: #4c7fea;
							color: #4c7fea;
						}
					}
				}
			}
		}

		.btn-box2 {
			position: fixed;
			bottom: 60rpx;
			left: 0;
			right: 0;
			margin: 0 20rpx 0 20rpx;
			flex-shrink: 0;
			width: auto;
			height: auto;
			// z-index: 9999;
			display: flex;
			// margin: 10rpx 0;
			flex-direction: row !important;
			align-items: center;
			justify-content: space-between;

			>view {
				width: 320rpx;
				height: 40px;
				border-radius: 20rpx;
				border: solid 1rpx #4c7fea;
				align-items: center;
				justify-content: center;
				// margin-bottom: 35rpx
			}

			.reset {
				margin-right: 10rpx;
				color: #4c7fea;
			}

			.submit {
				margin-left: 10rpx;
				color: #fff;
				background-color: #4c7fea;
			}
		}
	}

	.mask {
		z-index: 10;
		position: fixed;
		height: 100%;
		top: 0;
		left: 20rpx;
		right: 20rpx;
		bottom: 0;
		background-color: rgba(0, 0, 0, 0);
		transition: background-color .05s linear;
		overflow-y: hidden;

		&.show {
			background-color: rgba(50, 50, 50, 0.5);
		}

		&.hide {
			display: none;
		}
	}

	/* 字体图标 */
	@font-face {
		font-family: "HM-FD-font";
		src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAALAAAsAAAAABpQAAAJzAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDBgp4gQIBNgIkAwwLCAAEIAWEbQc5G8sFERWMIbIfCbbzqA4hp7InSBibVsYGb4J42o82b3e/nJlHMw/NHbGOlwKJRCRpwzPtpAECCOZubdqxjYpQLMlVg+70/08edrgQOtx2ukpVyApZn+dyehPoQObHo3O85rYx9vOjXoBxQIHugW2yIkqIW2QXcScu4jwE8CSWbKSmrqUHFwOaJoCsLM5P4haSGIxRcRHshrUGucLCVcfqI3AZfV/+USguKCwNmtsxVztDxU/n55C+3W0Z4QQpEOTNFqCBbMCAjDUWB9CIwWk87aa70cYgqLkyd3dEmm+18R8eKATEBrV7A5CulBT8dKiWOYZk412XNcDdKSEKSGODnyKIDl+dmVt9/Dx4pu/xyeutkMlHISGPTsPCnoTNP9nOT6wTtDdlO6dPr47efvj942lkYuQzrhMKEjq9N6y98P3340gmlJ/RStUD6F31CAEEPtUW94/7rf+7XgaAz57X0ZHXAGsFFwVgw38yALuMb0IBbVyNamFYEw4oKMDTj3AHRQP5Pt4dci9VwSVkRNQh5r7CLskZadhsWHhRDBsXczk8ZYk3ewnCxmQeQKa3BOHvA8XXO2j+vqRhf7CE+sPmn4anvoL29JLa4qqaUQkmoK+QG2osCckq7txi2leK86aIPyJ3eQZ8xytXYmyQ51jQndJAxIJlqiGSLsOqImiZCjTiZCJt6Lq26U2OoXqwUo0hRaAE0K5AziANy/uLVeXzWyjVqyjcoeupjxDr5MMDn8MDkLG9Aenu5ZrOSSoghAUsRmogkkahSoWAtnlUARnCkY3It0Iu7mWhdmd9Z/19BwBP6GidEi0G56opckXTGZVSPxgAAAA=');
	}

	.iconfont {
		font-family: "HM-FD-font" !important;
		font-size: 28rpx;
		font-style: normal;
		color: #757575;

		&.triangle {
			&:before {
				content: "\e65a";
			}
		}

		&.selected {
			&:before {
				content: "\e607";
			}
		}
	}
</style>