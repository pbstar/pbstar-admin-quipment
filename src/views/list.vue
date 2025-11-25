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
      :pagination="pagination"
      @paginationChange="toPageChange"
    >
      <template #age="scope">
        <span v-show="scope.row.age < 25">{{ scope.row.age }}</span>
        <span v-show="scope.row.age >= 25">{{ scope.row.age }}（老年人）</span>
      </template>
      <template #topLeft>
        <p-button type="primary" @click="toTopBtnClick({ btn: 'add' })"> 新增 </p-button>
      </template>
      <template #operation="{ row }">
        <p-button
          type="primary"
          size="small"
          link
          @click="toRightBtnClick({ row, btn: 'other' })"
        >
          其他
        </p-button>
      </template>
    </p-table>
  </div>
</template>
<script setup>
import { ref, onMounted } from "vue";
import { ElMessage } from "element-plus";
import request from "@Passets/utils/request";
import { PTable, PSearch, PTitle, PButton } from "@Pcomponents";
const data = ref([]);
const column = ref([
  { key: "name", label: "设备名称" },
  { key: "age", label: "服役年限", slot: "age" },
  { key: "sex", label: "性别" },
  { key: "ethnic", label: "民族", enumKey: "ethnic" },
  { key: "isHealthy", label: "是否健康", enumKey: "boolean" },
]);
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
  const sexOptions = [
    { label: "特种设备", value: "1" },
    { label: "非特种设备", value: "2" },
  ]
  tableRef.value.toChangeColumn([
    {
      key: "sex",
      options: sexOptions,
    },
  ]);
  searchRef.value.toChangeData([
    {
      key: "sex",
      options: sexOptions,
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
