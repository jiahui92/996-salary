<template>
	<view class="content">
		<view class="field-wrap">
			<view class="input-field">
				<view>月薪</view>
				<input type="number" v-model="salary" placeholder="月薪" />
			</view>
			<view class="input-field">
				<view>年终奖月数</view>
				<input type="number" v-model="yearEndAwards" placeholder="0.00" />
			</view>
			<view class="input-field">
				<view>每周工作天数</view>
				<input type="number" v-model="daysPerWeek" placeholder="0.00" />
			</view>
			<view class="input-field">
				<view>日均工作时长</view>
				<input type="number" v-model="hoursPerDay" placeholder="0.00" />
			</view>
		</view>
		
		<view class="result-base">
			<view>每周加班<text>{{overtimePerWeekCpt}}</text>小时，</view>
			<view>时薪<text>{{hourlyPayInLawCpt}}</text>元，年薪<text>{{annualSalaryCpt}}</text>元</view>
		</view>
		
		<view class="result-table">
			<view class="line header">
				<view></view>
				<view><view>每周</view>工作时长</view>
				<view><view>年薪</view>没加班费</view>
				<view><view>年薪</view>有加班费</view>
			</view>
			<view v-for="t in listCpt" :key="t.name">
				<view>{{t.name}}</view>
				<view>{{t.hoursPerWeek}}</view>
				<view>{{t.annualSalary}}</view>
				<view>{{t.annualSalaryInLaw}}</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			salary: 15000,
			yearEndAwards: 1,
			daysPerWeek: 5,
			hoursPerDay: 8,
		};
	},
	computed: {
		// 年薪
		annualSalaryCpt () {
			return this.salary * (12 + this.yearEndAwards * 1); // 一年的收入
		},
		// 时薪
		hourlyPayCpt () {
			const totalHoursPerYear = this.daysPerWeek * this.hoursPerDay * 52;
			return parseInt(this.annualSalaryCpt / totalHoursPerYearCpt) || 0;
		},
		// 时薪: 考虑加班费
		hourlyPayInLawCpt () {
			const totalHoursPerYear = this.getHoursPerWeekInLaw(this.hoursPerDay, this.daysPerWeek) * 52;
			return parseInt(this.annualSalaryCpt / totalHoursPerYear) || 0;
		},
		// 每周加班时长
		overtimePerWeekCpt () {
			return Math.max(0, this.daysPerWeek * this.hoursPerDay - 40);
		},
		listCpt () {
			const list = [
				{name: '955', hoursPerDay: 8, daysPerWeek: 5},
				{name: '965', hoursPerDay: 9, daysPerWeek: 5},
				{name: '995', hoursPerDay: 12, daysPerWeek: 5},
				// {name: '995.5', hoursPerDay: 12, daysPerWeek: 5.5},
				{name: '9115', hoursPerDay: 14, daysPerWeek: 5},
				// {name: '966', hoursPerDay: 9, daysPerWeek: 6},
				{name: '996', hoursPerDay: 12, daysPerWeek: 6},
				{name: '9116', hoursPerDay: 14, daysPerWeek: 6},
				{name: '007', hoursPerDay: 24, daysPerWeek: 7},
			];
			
			const handleMoney = (num) => `${(num / 10000).toFixed(1)}万`;
			
			return list.map((t) => {
				const hoursPerWeekInLaw = this.getHoursPerWeekInLaw(t.hoursPerDay, t.daysPerWeek);
				
				t.hoursPerWeek = t.hoursPerDay * t.daysPerWeek;
				t.annualSalary = t.hoursPerWeek * 52 * this.hourlyPayInLawCpt;
				t.annualSalaryInLaw = hoursPerWeekInLaw * 52 * this.hourlyPayInLawCpt;
				
				t.annualSalary = handleMoney(t.annualSalary);
				t.annualSalaryInLaw = handleMoney(t.annualSalaryInLaw);
				
				return t;
			});
		}
	},
	onLoad() {},
	methods: {
		// 算上加班补偿的总时长，比如周末加班2倍工资，等价于工作1小时-->2小时
		getHoursPerWeekInLaw (hoursPerDay, daysPerWeek) {
			// 补偿：日常超过8小时，按照1.5倍工作时间计算
			const fixDailyHours = Math.max(0, hoursPerDay - 8) * 5 * 0.5;
			// 补偿：周六日按照2倍工作时间计算
			const fixWeekendHours = Math.max(0, daysPerWeek - 5) * hoursPerDay * 1; 
			return hoursPerDay * daysPerWeek + fixDailyHours + fixWeekendHours;
		}
	}
};
</script>

<style lang="scss" scoped>
.content {
	padding: 10rpx;
}
	
.field-wrap {
	display: flex;
	flex-flow: row wrap;
	justify-content: space-around;
	
	.input-field {
		flex: 35%;
		padding: 18rpx 5%;
		text-align: right;
		
		view, input {
			padding-right: 10rpx;
		}
		
		input {
			font-size: 60rpx;
			font-weight: bold;
			border-bottom: 1rpx solid grey;
		}
	}
}

.result-base {
	margin: 80rpx 30rpx;
	text {
		font-weight: bold;
		font-size: 50rpx;
		margin: 0 5rpx;
		text-decoration: underline;
	}
}

.result-table {
	& > view {
		display: flex;
		view {
			text-align: center;
			&:nth-child(1) { flex: 1; font-weight: bold; }
			&:nth-child(2) { flex: 2; }
			&:nth-child(3) { flex: 2; }
			&:nth-child(4) { flex: 2; }
		}
	}
	
	.header {
		font-weight: bold;
	}
}
</style>
