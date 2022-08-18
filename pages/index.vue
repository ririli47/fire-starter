<template>
  <div>
    <div>
        <h3>今現在の情報</h3>
        <div class="flex-parent">
            <label class="label-width">総資産額</label>
            <input
                v-model="totalMoney"
                required
                type="number"
                min="0"
            >
        </div>
        <div class="flex-parent">
            <label class="label-width">生活防衛資金</label>
            <input
            v-model="defanseMoney"
            required
            type="number"
            min="0"
            >
        </div>
    </div>
    <div>
        <h3>投資計画の情報</h3>
        <div class="flex-parent">
            <label class="label-width">想定投資利率/年</label>
            <input
                v-model="investmentInterestRate"
                required
                type="number"
                min="0"
            >
        </div>
        <div class="flex-parent">
            <label class="label-width">追加入金/月</label>
            <input
                v-model="additionalMoney"
                required
                type="number"
                min="0"
            >
        </div>
    </div>
    <p>年換算：{{ additionalMoneyPerYear == undefined ? '0' : additionalMoneyPerYear.toLocaleString()}} 円</p>

    <div>
        <h3>FIREプランの情報</h3>
        <div class="flex-parent">
            <label class="label-width">基準年間生活費</label>
            <input
                v-model="costOfLiving"
                required
                type="number"
                min="0"
            >
        </div>
        <div class="flex-parent">
            <label class="label-width">切り崩し（または配当）率/年</label>
            <input
            v-model="useMoneyRate"
            required
            type="number"
            min="0"
            >
        </div>
    </div>
    <p>FIRE達成目標金額：{{ targetCompleteFire == undefined ? '0' : targetCompleteFire.toLocaleString()}} 円</p>

    <div>
        <h3>試算オプション</h3>
        <div>
            <label class="label-width">試算年数</label>
            <input
                v-model="calcYears"
                label="試算年数"
                required
                type="number"
                min="1"
            >
        </div>
        <div>
            <input
                type="checkbox"
                v-model="stopAdditionalMoney"
            >FIRE達成後に入金を止める
        </div>
        <div>
            <input
                type="checkbox"
                v-model="startUseMoney"
            >FIRE達成後に切り崩し率で切り崩し始める
        </div>
    </div>
    <input
    type="button"
    @click="reset()"
    value="ResetForm"
    >
    <input
    type="button"
    @click="createCharts()"
    value="グラフ描画"
    >
    <BarChart ref="barChartRef" :calculationResult="calculationResult" :target="targetCompleteFire" :calcYears="calcYears"/>
  </div>
</template>

<script setup>
  import BarChart from '@/components/BarChart.vue'
  const barChartRef = ref();

  // data
  let stopAdditionalMoney = ref(false);
  let startUseMoney = ref(false);
  let calcYears = ref(1);
  let totalMoney = ref(0);
  let defanseMoney = ref(0);
  let costOfLiving = ref(0);
  let useMoneyRate = ref(0);
  let investmentInterestRate = ref(0);
  let additionalMoney = ref(0);
  const additionalMoneyPerYear = computed(() => additionalMoney.value * 12)

  // main logic
  let calculationResult = ref();
  calculationResult = computed(() => {
    let completeFlag = false
    let result = [0];
    result[0] = totalMoney.value; // 0年目
    let nextVal = totalMoney.value - defanseMoney.value; // 0年目      
    for (let i=1; i<calcYears.value; i++) {
      if (completeFlag && (stopAdditionalMoney.value || startUseMoney.value)) {
        // 次年度額 = (総資産額-生活防衛資金) *(1+利率)
        nextVal = nextVal * ((100 + investmentInterestRate.value) / 100)
        if (startUseMoney.value) {
          // -= 年切り崩し額
          nextVal -= (nextVal*(useMoneyRate.value/100))
        }
        if (!stopAdditionalMoney.value) {
          // += 追加投資額
          nextVal += additionalMoneyPerYear.value
        }
        result.push(parseInt((defanseMoney.value + nextVal).toFixed()))
      }
      else {
        // 次年度額 = (総資産額-生活防衛資金) *(1+利率) + 追加投資額
        nextVal = nextVal * ((100 + investmentInterestRate.value) / 100) + additionalMoneyPerYear.value
        result.push(parseInt((defanseMoney.value + nextVal).toFixed().toLocaleString()))
      }
      if (nextVal > targetCompleteFire.value) {
        completeFlag = true;
      }
    }
    return result;
  })
  let targetCompleteFire = computed(() => {
    return useMoneyRate.value == 0 ? '-' : (costOfLiving.value / ((useMoneyRate.value)/100))
  })
  
  // methods
  const reset = () => {
    totalMoney.value = 0
    defanseMoney.value = 0
    costOfLiving.value = 0
    investmentInterestRate.value = 0
    additionalMoney.value = 0
    useMoneyRate.value = 0
    calcYears.value = 1
    startUseMoney.value = false
    stopAdditionalMoney.value = false
  }
  const createCharts = () => {
    barChartRef.value.createCharts();
  }
</script>

<style scoped>
.flex-parent {
    display: flex;
}
.label-width {
    width: 30vw;
}
</style>