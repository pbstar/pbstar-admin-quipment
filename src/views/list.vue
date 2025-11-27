<template>
  <div class="page">
    <p-title :list="['设备台账']"></p-title>
    <p-search style="margin-top: 10px" @search="toSearch" @reset="toReset">
      <p-item
        class="item"
        :config="{ label: '设备名称', type: 'input' }"
        v-model="searchValue.name"
      />
      <p-item
        class="item"
        :config="{ label: '服役年限', type: 'inputNumber' }"
        v-model="searchValue.age"
      />
      <p-item
        class="item"
        :config="{ label: '性别', type: 'select', options: sexOptions }"
        v-model="searchValue.sex"
      />
      <p-item
        class="item"
        :config="{
          label: '是否健康',
          type: 'select',
          enumKey: 'boolean',
        }"
        v-model="searchValue.isHealthy"
      />
    </p-search>
    <p-table
      style="margin-top: 10px"
      ref="tableRef"
      :data="data"
      :pagination="pagination"
      @paginationChange="toPageChange"
    >
      <template #column>
        <el-table-column prop="name" label="设备名称" />
        <el-table-column prop="age" label="服役年限">
          <template #default="{ row }">
            <span v-show="row.age < 25">{{ row.age }}</span>
            <span v-show="row.age >= 25">{{ row.age }}（老年人）</span>
          </template>
        </el-table-column>
        <el-table-column prop="sex" label="性别">
          <template #default="{ row }">
            {{ getSexLabel(row.sex) }}
          </template>
        </el-table-column>
        <el-table-column prop="ethnic" label="民族">
          <template #default="{ row }">
            {{ enumStore.getEnumLabel("ethnic", row.ethnic) }}
          </template>
        </el-table-column>
        <el-table-column prop="isHealthy" label="是否健康">
          <template #default="{ row }">
            {{ enumStore.getEnumLabel("boolean", row.isHealthy) }}
          </template>
        </el-table-column>
        <el-table-column
          prop="operation"
          label="操作"
          fixed="right"
          width="120"
        >
          <template #default="{ row }">
            <p-button
              type="primary"
              size="small"
              link
              @click="handleOther(row)"
            >
              其他
            </p-button>
          </template>
        </el-table-column>
      </template>
      <template #topLeft>
        <p-button type="primary" @click="handleAdd()"> 新增 </p-button>
      </template>
    </p-table>
  </div>
</template>
<script setup>
import { ref, onMounted } from "vue";
import { ElMessage } from "element-plus";
import request from "@Passets/utils/request";
import { PTable, PSearch, PTitle, PButton, PItem } from "@Pcomponents";
import { useEnumStore } from "@Passets/stores/enum";
const data = ref([]);
const enumStore = useEnumStore();
const tableRef = ref(null);
const pagination = ref({
  pageNumber: 1,
  pageSize: 10,
  total: 0,
});
const searchValue = ref({});
const sexOptions = ref([
  { label: "特种设备", value: "1" },
  { label: "非特种设备", value: "2" },
]);

// 根据 value 获取 label
const getSexLabel = (value) => {
  const option = sexOptions.value.find((item) => item.value === value);
  return option ? option.label : value;
};

onMounted(async () => {
  // 预加载枚举数据
  await enumStore.getEnum("ethnic,boolean");
  initTable();
});
const toSearch = () => {
  pagination.value.pageNumber = 1;
  initTable();
};
const toReset = () => {
  searchValue.value = {};
  toSearch();
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
const handleAdd = () => {
  ElMessage.warning(
    "此页面为外部子应用测试页面，禁止新增，如需新增请前往示例应用",
  );
};
const handleOther = (row) => {
  ElMessage.success("其他");
};
</script>
<style lang="scss" scoped>
.page {
  width: 100%;
  padding: 0 10px;
  background-color: var(--c-bg);

  .item {
    width: 250px;
    margin-bottom: 10px;
    margin-right: 10px;
  }
}
</style>
