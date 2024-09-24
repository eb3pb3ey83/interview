<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input
          v-model="tempData.name"
          label="姓名"
          :rules="[(val) => !!val || '姓名不得空白']"
          ref="nameInput"
        />
        <q-input
          v-model="tempData.age"
          label="年龄"
          type="number"
          :rules="ageRules"
          ref="ageInput"
        />
        <q-btn
          color="primary"
          label="新增"
          @click="handleSubmit"
          v-if="!isUpdating"
        />
        <q-btn
          color="warning"
          label="更新"
          @click="handleSubmit"
          v-if="isUpdating"
        />
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { onMounted, ref, computed } from 'vue';

interface btnType {
  label: string;
  icon: string;
  status: 'edit' | 'delete';
}
interface Data {
  id: string;
  age: string | number;
  name: string;
}

const baseApiUrl = 'https://dahua.metcfire.com.tw/api/CRUDTest';
const ageRules = [
  (val: string) => val !== '' || '年齡不得空白',
  (val: string) => /^\d+$/.test(val) || '年齡必須是正整數',
];
const nameInput = ref<any>(null);
const ageInput = ref<any>(null);
const blockData = ref<Data[]>([]);
const tableConfig = ref<QTableProps['columns']>([
  { label: '姓名', name: 'name', field: 'name', align: 'left' },
  { label: '年齡', name: 'age', field: 'age', align: 'left' },
]);

const tableButtons = ref<btnType[]>([
  { label: '編輯', icon: 'edit', status: 'edit' },
  { label: '刪除', icon: 'delete', status: 'delete' },
]);

const tempData = ref<Data>({ name: '', age: '', id: '' });
const updateId = ref('');
const isUpdating = computed(() => updateId.value !== '');

// 重置表單
const resetForm = () => {
  tempData.value = { name: '', age: '', id: '' };
  updateId.value = '';
  nameInput.value.resetValidation();
  ageInput.value.resetValidation();
};

// 通用的 API 調用函數
const apiCall =
  (method: 'post' | 'patch' | 'delete', url: string) => async (data?: any) => {
    try {
      await axios({ method, url, data });
      await fetchData();
      if (method !== 'delete') resetForm();
    } catch (error) {
      console.error(`${method.toUpperCase()} 失敗:`, error);
    }
  };

const addRow = apiCall('post', baseApiUrl);
const updateRow = apiCall('patch', baseApiUrl);
const deleteRow = (id: string) => apiCall('delete', `${baseApiUrl}/${id}`)();

const handleSubmit = () => {
  if (!nameInput.value || !ageInput.value) return;
  const isNameValid = nameInput.value.validate();
  const isAgeValid = ageInput.value.validate();

  if (isNameValid && isAgeValid) {
    const dataToSend = {
      ...tempData.value,
      age: Number(tempData.value.age),
    };
    (isUpdating.value ? updateRow : addRow)(dataToSend);
  }
};

// 獲取資料
const fetchData = async () => {
  try {
    const response = await axios.get(
      'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    );
    blockData.value = response.data;
  } catch (error) {
    console.error('資料獲取失敗:', error);
  }
};

// 編輯資料
const editRow = (row: Data) => {
  tempData.value = { ...row, age: String(row.age) };
  updateId.value = row.id;
};

const actionMap = {
  edit: editRow,
  delete: (data: Data) => deleteRow(data.id),
};

const handleClickOption = (btn: btnType, data: Data) => {
  const action = actionMap[btn.status];
  action && action(data);
};

onMounted(fetchData);
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
