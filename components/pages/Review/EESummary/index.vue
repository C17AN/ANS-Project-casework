<template>
  <div class="Container">
    <div class="table-container">
      <div class="header">
        <h2 class="data-title">
          {{ dataTypeheaderText }}
        </h2>
      </div>
      <div class="result-table">
        <SummaryTable />
        <LineChart />
      </div>
    </div>
    <ul class="button__list">
      <button @click="excelDown">데이터 저장</button>
      <button @click="handleRoute('/review')">돌아가기</button>
    </ul>
  </div>
</template>

<script>
import { xlsx } from "xlsx";
import { mapState } from "vuex";
import { YEAR, MAX } from "@/utils/constants/config.js";

export default {
  name: "ReviewTable",
  components: {
    SummaryTable: () => import("./EESummaryTable.vue"),
    LineChart: () => import("./EELineChart.vue"),
  },
  props: {
    tableToDisplay: {
      type: String,
      value: "",
    },
  },
  methods: {
    async excelDown() {
      const xlsx = require("xlsx");
      // 엑셀 파일 생성
      const book = xlsx.utils.book_new();
      // data get > 실 개발시 api 호출
      const EESummaryData = await this.getEESummaryData();
      // sheet 생성 - aoa_to_sheet 방식
      const worksheet = xlsx.utils.aoa_to_sheet(EESummaryData);
      // 엑셀 파일에 sheet set(엑셀파일, 시트데이터, 시트명)
      xlsx.utils.book_append_sheet(book, worksheet, "총 기대효과");
      // 엑셀 다운로드
      xlsx.writeFile(book, "국가 항행계획 총 기대효과.xlsx");
    },
    // aoa는 행열 데이터를 엑셀과 동일하게 get
    getEESummaryData() {
      let arr = [];
      const Year = Array(YEAR).fill(0);
      const N_Flight = Array(YEAR).fill(0);
      const FTR_EE = Array(YEAR).fill(0);
      const USER_EE = Array(YEAR).fill(0);
      const SAFETY_EE = Array(YEAR).fill(0);
      const Total_EE = Array(YEAR).fill(0);
      const ACCUM_EE = Array(YEAR).fill(0);

      for (let t = 0; t < YEAR; t++) {
        Year[t] = new Date().getFullYear() + t;
      }

      for (let l = 0; l < MAX; l++) {
        for (let t = 0; t < YEAR; t++) {
          N_Flight[t] =
            N_Flight[t] +
            this.ANSDataTemplate.N_DD_Flght.value[l][t] +
            this.ANSDataTemplate.N_AD_Flght.value[l][t] +
            this.ANSDataTemplate.N_AI_Flght.value[l][t] +
            this.ANSDataTemplate.N_DI_Flght.value[l][t];
        }
      }

      for (let l = 0; l < MAX; l++) {
        for (let t = 0; t < YEAR; t++) {
          FTR_EE[t] =
            FTR_EE[t] +
            this.ANSDataTemplate.CER_DDcost.value[l][t] +
            this.ANSDataTemplate.CER_DIcost.value[l][t] +
            this.ANSDataTemplate.CER_ADcost.value[l][t] +
            this.ANSDataTemplate.CER_AIcost.value[l][t] +
            this.ANSDataTemplate.CER_DRcost.value[l][t] +
            this.ANSDataTemplate.CER_DIRcost.value[l][t] +
            this.ANSDataTemplate.CER_AIRcost.value[l][t] +
            this.ANSDataTemplate.FR_DDcost.value[l][t] +
            this.ANSDataTemplate.FR_DIcost.value[l][t] +
            this.ANSDataTemplate.FR_ADcost.value[l][t] +
            this.ANSDataTemplate.FR_AIcost.value[l][t] +
            this.ANSDataTemplate.FR_DRcost.value[l][t] +
            this.ANSDataTemplate.FR_DIRcost.value[l][t] +
            this.ANSDataTemplate.FR_AIRcost.value[l][t] +
            this.ANSDataTemplate.OPR_DDcost.value[l][t] +
            this.ANSDataTemplate.OPR_DIcost.value[l][t] +
            this.ANSDataTemplate.OPR_AIcost.value[l][t];
        }
      }
      for (let l = 0; l < MAX; l++) {
        for (let t = 0; t < YEAR; t++) {
          USER_EE[t] =
            USER_EE[t] +
            this.ANSDataTemplate.CER_DDcost_byADLY.value[l][t] +
            this.ANSDataTemplate.CER_DIcost_byADLY.value[l][t] +
            this.ANSDataTemplate.CER_ADcost_byADLY.value[l][t] +
            this.ANSDataTemplate.CER_AIcost_byADLY.value[l][t] +
            this.ANSDataTemplate.FR_DDcost_byADLY.value[l][t] +
            this.ANSDataTemplate.FR_DIcost_byADLY.value[l][t] +
            this.ANSDataTemplate.FR_ADcost_byADLY.value[l][t] +
            this.ANSDataTemplate.FR_AIcost_byADLY.value[l][t] +
            this.ANSDataTemplate.OPR_ADcost_DLY.value[l][t] +
            this.ANSDataTemplate.OPR_AIcost_DLY.value[l][t] +
            this.ANSDataTemplate.BNF_AD_PSG.value[l][t] +
            this.ANSDataTemplate.BNF_AI_PSG.value[l][t];
        }
      }

      for (let l = 0; l < MAX; l++) {
        for (let t = 0; t < YEAR; t++) {
          SAFETY_EE[t] =
            SAFETY_EE[t] + this.ANSDataTemplate.Safty_cost.value[t];
        }
      }

      for (let t = 0; t < YEAR; t++) {
        Total_EE[t] = FTR_EE[t] + USER_EE[t] + SAFETY_EE[t];
      }

      for (let t = 0; t < YEAR; t++) {
        if (t == 0) {
          ACCUM_EE[t] = Total_EE[t];
        } else {
          ACCUM_EE[t] = Total_EE[t] + ACCUM_EE[t - 1];
        }
      }
      var x = 0;
      for (let t = 0; t < 18; t++) {
        x = x + SAFETY_EE[t];
      }

      arr.push([
        "연도",
        "총 운항편수",
        "운항효율성 기대효과",
        "정시성 기대효과",
        "안전운항 기대효과",
        "총 기대효과",
        "누적 기대효과",
      ]);

      for (let i = 0; i < YEAR; i++) {
        arr.push([
          Year[i],
          N_Flight[i],
          FTR_EE[i],
          USER_EE[i],
          SAFETY_EE[i],
          Total_EE[i],
          ACCUM_EE[i],
        ]);
      }

      return arr;
    },
    handleRoute(path) {
      this.$router.push(path);
    },
    Save() {
      this.tableToDisplay = "EESummaryTable";
    },
  },
  computed: {
    ...mapState("ansData", ["ANSDataTemplate"]),
    dataTypeheaderText() {
      const filteredDataName = "국가 항행계획 기대효과(개별사업)";
      return this.filteredDataName || filteredDataName;
    },
  },
};
</script>

<style lang="scss">
.table-container {
  display: flex;
  flex-direction: column;
  position: relative;
  flex: 1;
  border: 1px solid #cdcdcd;
  border-radius: 0.5rem;
  box-shadow: 4px 4px 4px rgba(0, 0, 0, 0.1);
  overflow-x: scroll;
}

.table-container::after {
  content: "";
  background: url("~/assets/images/intro-logo.png") no-repeat;
  background-position: center;
  opacity: 0.12;
  position: absolute;
  top: 8px;
  left: 0px;
  right: 0px;
  bottom: 0px;
  z-index: -1;
}

.header {
  display: flex;
  position: relative;
  align-items: center;
  justify-content: space-between;
  padding: 1.5rem 1.5rem;
  font-size: 1.125rem;
  font-weight: bold;
  top: 0;
  background-color: white;
  border-bottom: 1px solid #cdcdcd;
}

.data-title {
  width: 100%;
  text-align: center;
}

.data-save {
  color: #ababab;
  position: absolute;
  right: 1rem;
  font-size: 0.875rem;
  white-space: nowrap;
  cursor: pointer;
  transition: 0.2s ease-in-out all;
  padding: 0.375rem 0.5rem;
}

.data-save:hover {
  color: #333;
  background-color: #efefef;
  border-radius: 0.5rem;
}

.result-table {
  display: flex;
  flex-direction: column;
  padding: 1rem;
  height: 100%;
  overflow-x: hidden;
}
.Container {
  width: 95%;
  height: 60%;
  display: flex;
  gap: 2rem;
  padding: 1rem;
  font-size: 100%;
}
</style>
