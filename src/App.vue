<style>
* {
	font-family: "Courier New", Courier, monospace;
}

body {
	display: flex;
	flex-direction: column;
	align-items: center;
}

h1 {
	text-align: center;
	margin-top: 50px;
}

#date-selector {
	width: 800px;
	max-width: 80vw;
	margin-top: 50px;

	display: grid;
	grid-template-columns: 1fr 1fr 1fr;
	gap: 20px;
}

#date-selector .select-group {
	display: flex;
	flex-direction: column;
}

#date-selector .select-group label {
	font-weight: bold;
	margin-bottom: 10px;
}

#date-selector .select-group select {
	padding: 10px;
}

#result {
	text-align: center;
	margin-top: 50px;
}

footer {
	text-align: center;
	margin-top: 50px;
}
footer a {
	text-decoration: none;
	color: #999;
	transition: color .3s ease;
	font-size: 0.8em;
}
footer a:hover {
	color: rgb(60, 60, 255);
}

@media (max-width: 800px) {
	h1 { font-size: 5vw }
}
@media (max-width: 500px) {
	#date-selector { grid-template-columns: 1fr }
	h2 { font-size: 5vw }
	h3 { font-size: 4vw }
}
</style>

<template>
	<div id="app">
		<h1>How Many Days Until...</h1>

		<div id="date-selector">
			<div id="year" class="select-group">
				<select name="year" id="year" v-model="selectedYear" @change="selectedMonth = ''; selectedDay = ''">
					<option value="">Year...</option>
					<option v-for="index in 20" :key="index">
						{{ new Date().getFullYear() + index - 1 }}
					</option>
				</select>
			</div>

			<div id="month-group" class="select-group" v-if="selectedYear != ''">
				<select name="month" id="month" v-model="selectedMonth" @change="selectedDay = ''">
					<option value="">Month...</option>
					<option
						v-for="index in 12"
						:key="index"
						:disabled="hasMonthPassed(index-1)"
						:value="(index-1).toString()"
					>
						{{ Object.keys(monthNames)[index-1] }}
						{{ hasMonthPassed(index-1) ? "(PASSED)" : "" }}
					</option>
				</select>
			</div>

			<div id="day-group" class="select-group" v-if="selectedYear && selectedMonth">
				<select name="day" id="day" v-model="selectedDay">
					<option value="">Day...</option>
					<option
						v-for="index in daysInSelectedMonth"
						:key="index"
						:disabled="hasDayPassed(index)"
					>
						{{ index }}
						{{ hasDayPassed(index) ? "(PASSED)" : "" }}
					</option>
				</select>
			</div>
		</div>

		<div id="result" v-if="selectedMonth && selectedYear">
			<h2>{{ formattedFromNow }}</h2>
			<h3 v-if="daysLeftFromNow >= 27">({{ daysLeftFromNow }} days)</h3>
		</div>
		
		<footer>
			<a href="https://berkinakkaya.github.io/" target="_blank">
				berkinakkaya.github.io
			</a>
		</footer>
	</div>
</template>

<script>
import moment from "moment";

export default {
	name: "App",
	data() {
		return {
			selectedDay: "",
			selectedMonth: "",
			selectedYear: "",
			monthNames: {
				January: "01",
				February: "02",
				March: "03",
				April: "04",
				May: "05",
				June: "06",
				July: "07",
				August: "08",
				September: "09",
				October: "10",
				November: "11",
				December: "12",
			},
		};
	},
	computed: {
		daysInSelectedMonth() {
			const { selectedYear, selectedMonth } = this;
			return new Date(selectedYear, selectedMonth, 0).getDate();
		},
		formattedFromNow() {
			const currentMonth = new Date().getMonth();
			if (!this.selectedDay && this.selectedMonth == currentMonth) {
				return "";
			}

			if (this.daysLeftFromNow <= 26) {
				const plural = this.daysLeftFromNow > 1 ? "s" : "";
				return this.daysLeftFromNow + " day" + plural + " left";
			}

			const { selectedYear, selectedMonthIndex, selectedDay } = this;
			const YYYYMMDD = `${selectedYear}${selectedMonthIndex}${selectedDay}`;
			return moment(YYYYMMDD, "YYYYMMDD").fromNow();
		},
		daysLeftFromNow() {
			const { selectedYear, selectedMonth, selectedDay } = this;
			const day = parseInt(selectedDay) + 1 || "1";
			const target = new Date(selectedYear, selectedMonth, day);
			return Math.round((target - new Date()) / (1000 * 60 * 60 * 24));
		},
		selectedMonthIndex() {
			const name = Object.keys(this.monthNames)[this.selectedMonth];
			return this.monthNames[name];
		},
	},
	methods: {
		hasMonthPassed(month) {
			return new Date(this.selectedYear, month + 1, 1) - new Date() < 0;
		},
		hasDayPassed(day) {
			const { selectedYear, selectedMonth } = this;
			return new Date(selectedYear, selectedMonth, day) - new Date() < 0;
		},
	},
};
</script>