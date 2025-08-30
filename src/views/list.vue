<template>
  <div class="page">
    <p-title :list="['设备台账']"></p-title>
    <p-search
      style="margin-top: 10px"
      ref="searchRef"
      :data="searchData"
      @btnClick="toSearch"
    ></p-search>
    <p-table
      style="margin-top: 10px"
      ref="tableRef"
      :data="data"
      :column="column"
      :topBtn="topBtn"
      :rightBtn="rightBtn"
      tableKey="table1"
      showSetting
      :pagination="pagination"
      @paginationChange="toPageChange"
      @topBtnClick="toTopBtnClick"
      @rightBtnClick="toRightBtnClick"
    >
      <template #age="scope">
        <span v-show="scope.row.age < 25">{{ scope.row.age }}</span>
        <span v-show="scope.row.age >= 25">{{ scope.row.age }}（老年人）</span>
      </template>
    </p-table>
  </div>
</template>
<script setup>
import { ref, onMounted } from "vue";
import { ElMessage } from "element-plus";
import request from "@Passets/utils/request";
import PTable from "@Pcomponents/base/p-table/index.vue";
import PSearch from "@Pcomponents/base/p-search/index.vue";
import PTitle from "@Pcomponents/base/p-title/index.vue";
const data = ref([]);
const column = ref([
  { key: "name", label: "设备名称" },
  { key: "age", label: "服役年限", slot: "age" },
  { key: "sex", label: "性别" },
  { key: "ethnic", label: "民族", enumKey: "ethnic" },
  { key: "isHealthy", label: "是否健康", enumKey: "boolean" },
]);
const topBtn = ref([{ key: "add", label: "新增", auth: "list_add" }]);
const rightBtn = ref([{ key: "other", label: "其他" }]);
const tableRef = ref(null);
const pagination = ref({
  pageNumber: 1,
  pageSize: 10,
  total: 0,
});
const searchRef = ref(null);
const searchData = ref([
  { key: "name", label: "设备名称", type: "input" },
  { key: "age", label: "服役年限", type: "inputNumber" },
  { key: "sex", label: "性别", type: "select", options: [] },
  {
    key: "isHealthy",
    label: "是否健康",
    type: "select",
    enumKey: "boolean",
  },
]);
const searchValue = ref({});

onMounted(() => {
  tableRef.value.toChangeColumn([
    {
      key: "sex",
      options: [
        { label: "男", value: "1" },
        { label: "女", value: "2" },
      ],
    },
  ]);
  searchRef.value.toChangeData([
    {
      key: "sex",
      options: [
        { label: "男", value: "1" },
        { label: "女", value: "2" },
      ],
    },
  ]);
  initTable();
});
const toSearch = ({ data }) => {
  searchValue.value = data;
  initTable();
};
const toPageChange = ({ pageNumber, pageSize }) => {
  pagination.value.pageNumber = pageNumber;
  pagination.value.pageSize = pageSize;
  initTable();
};
const initTable = () => {
  request
    .post({
      url: "/example/person/getList",
      data: { ...searchValue.value, ...pagination.value },
    })
    .then((res) => {
      if (res && res.code == 200) {
        data.value = res.data.list;
        pagination.value.total = res.data.total;
      } else {
        ElMessage.error(res.msg || "操作异常");
      }
    });
};
const toTopBtnClick = ({ btn }) => {
  if (btn == "add") {
    ElMessage.warning(
      "此页面为外部子应用测试页面，禁止新增，如需新增请前往示例应用"
    );
  }
};
const toRightBtnClick = ({ btn, row }) => {
  if (btn == "other") {
    ElMessage.success("其他");
  }
};
</script>
<style lang="scss" scoped>
.page {
  width: 100%;
  padding: 0 10px;
  background-color: var(--c-bg);
}
</style>
