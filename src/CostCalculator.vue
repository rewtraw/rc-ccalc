<template>
	<div id="component">
		<div style="margin: 20px;"></div>
		<el-form :label-position="'right'" label-width="200px" :model="formLabelAlign">
			<el-form-item label="Call Type">
				<el-select v-model="type" placeholder="Select">
					<el-option v-for="item in types" :key="item" :label="item" :value="item">
					</el-option>
				</el-select>
			</el-form-item>
			<el-form-item label="Number of Leads">
				<el-input-number v-model="leads" :step="100"></el-input-number>
			</el-form-item>
			<el-form-item v-if="type == 'RoboCall' || type == 'RVM' || type == 'Transfer'" label="Audio Length (in Seconds)">
				<el-input-number v-model="length" :step="5"></el-input-number>
			</el-form-item>
			<el-form-item v-if="type == 'PushPoll'" label="Leave VM">
				<el-switch v-model="leave_vm"></el-switch>
			</el-form-item>
			<!-- <el-form-item v-if="type == 'PushPoll' || type == 'Survey'" label="Alaska">
				<el-switch v-model="alaska"></el-switch>
			</el-form-item> -->
			<el-form-item>
				<el-button type="primary" @click="calculate()">Calculate</el-button>
				<el-button @click="reset()">Reset</el-button>
			</el-form-item>
			<el-form-item label="Cost">
				<el-input placeholder="Click Calculate" v-model="cost" :disabled="true" style="width: 180px;"></el-input>
			</el-form-item>
			<el-form-item v-if="error" label="">
                    <el-alert style="width: 180px;"
                    :title="error"
                    type="error"
                    :closable="false">
                  </el-alert>
                
			</el-form-item>
		</el-form>
	</div>
</template>
<script>
	export default {
		methods: {
			calculate() {
				this.multi = 0;
				this.cost = '';
				this.error = '';
				if (this.leads <= 100000) {
					this.size = 'small'
				} else if (this.leads <= 500000) {
					this.size = 'medium'
				} else {
					this.size = 'large'
				}
				if (this.type == 'RoboCall' || this.type == 'RVM' || this.type == 'Transfer') {
					var effective_type = 'RoboCall'
				} else {
					var effective_type = this.type
				}
				for (var i = 0; i < this.pricing.length; i++) {
					var element = this.pricing[i]
					if (effective_type == element.type && (this.length <= element.maxlen && this.length >= element.minlen)) {
						this.multi = element.cost[this.size]
					} else if (effective_type == element.type && (this.length > 90)) {
						var additional_sec = this.length - 75;
						this.multi += (this.addl_cost.extra_15s * (1 + Math.floor(additional_sec / 15)));
						// console.log('Math.floor(additional_sec / 15): ', 1 + Math.floor(additional_sec / 15))
						// console.log('over 90s')
					} else if (effective_type == element.type && !element.minlen) {
						this.multi = element.cost[this.size]
					}
				}

				// console.log('base multi: ', this.multi)
				if (this.type == 'RVM') {
					this.multi += this.addl_cost.rvm;
				}
				if (this.type == 'Transfer') {
					this.multi += this.addl_cost.transfer;
				}
				if ((this.type == 'PushPoll' || this.type == 'Survey') && this.alaska) {
					this.multi += this.addl_cost.alaska;
				}
				if (this.type == 'PushPoll' && this.leave_vm) {
					this.multi += this.addl_cost.leave_vm;
				}
				// console.log('total multi: ', this.multi)
				// console.log('this.leads * this.multi: ', this.leads, ' * ', this.multi)
				this.cost = this.leads * this.multi;
				if ((this.type == 'RoboCall' || this.type == 'Transfer') && this.cost < 30) {
					this.cost = 30
					this.error = 'Minimum is $' + this.cost
				}
				if ((this.type == 'PushPoll' || this.type == 'RVM' || this.type == 'SMS') && this.cost < 50) {
					this.cost = 50
					this.error = 'Minimum is $' + this.cost
				}
				if ((this.type == 'Survey') && this.cost < 100) {
					this.cost = 100
					this.error = 'Minimum is $' + this.cost
				}
				this.cost = '$' + this.cost.toFixed(2).replace(/(\d)(?=(\d{3})+\.)/g, '$1,');
			},
			reset() {
				this.type = 'RoboCall';
				this.leads = null;
				this.length = null;
				this.cost = null;
				this.alaska = false;
				this.leave_vm = false;
				this.error = '';
			}
		},
		data() {
			return {
				error: '',
				labelPosition: 'right',
				formLabelAlign: {
					name: '',
					region: '',
					type: ''
				},
				msg: 'RoboCent Cost Calculator',
				leads: 1000,
				length: 15,
				cost: null,
				alaska: false,
				leave_vm: false,
				types: ['RoboCall', 'RVM', 'Transfer', 'PushPoll', 'Survey', 'SMS'],
				type: 'RoboCall',
				multi: null,
				pricing: [{
					type: 'RoboCall',
					minlen: 0,
					maxlen: 15,
					cost: {
						small: 0.01,
						medium: 0.01,
						large: 0.01
					}
				}, {
					type: 'RoboCall',
					minlen: 16,
					maxlen: 30,
					cost: {
						small: 0.02,
						medium: 0.018,
						large: 0.015
					}
				}, {
					type: 'RoboCall',
					minlen: 31,
					maxlen: 45,
					cost: {
						small: 0.03,
						medium: 0.026,
						large: 0.02
					}
				}, {
					type: 'RoboCall',
					minlen: 46,
					maxlen: 60,
					cost: {
						small: 0.04,
						medium: 0.034,
						large: 0.025
					}
				}, {
					type: 'RoboCall',
					minlen: 61,
					maxlen: 75,
					cost: {
						small: 0.05,
						medium: 0.044,
						large: 0.035
					}
				}, {
					type: 'RoboCall',
					minlen: 76,
					maxlen: 90,
					cost: {
						small: 0.06,
						medium: 0.054,
						large: 0.045
					}
				}, {
					type: 'PushPoll',
					cost: {
						small: 0.03,
						medium: 0.028,
						large: 0.026
					}
				}, {
					type: 'Survey',
					cost: {
						small: 0.035,
						medium: 0.033,
						large: 0.031
					}
				}, {
					type: 'SMS',
					cost: {
						small: 0.16,
						medium: 0.16,
						large: 0.16
					}
				}],
				addl_cost: {
					rvm: 0.05,
					transfer: 0.015,
					leave_vm: 0.015,
					alaska: 0.02,
					extra_15s: 0.01
				}
			}
		}
	}

</script>
<style>


</style>
