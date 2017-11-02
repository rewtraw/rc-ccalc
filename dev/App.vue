<template>
	<div id="component">
		<cost-calculator></cost-calculator>
	</div>
</template>
<script>
	import CostCalculator from '../src/index'
	export default {
		components: {
			CostCalculator
		},
		data() {
			return {
				msg: 'Hello'
			}
		}
	}

</script>


<style lang="scss">
	#component {
		font-family: 'Avenir', Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		color: #2c3e50;
	}

	body {
		margin: 0px;
		font-family: 'Avenir', Helvetica, Arial, sans-serif;
	}

</style>
