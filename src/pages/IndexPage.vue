<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" type="number" />
        <q-btn @click="createOrUpdateData" color="primary" class="q-mt-md"
          >新增/更新</q-btn
        >
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="tableConfig"
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
            <q-td v-for="col in props.cols" :key="col.name" :props="props">
              <div>{{ props.row[col.field] }}</div>
            </q-td>
            <q-td class="text-right" auto-width>
              <q-btn
                @click="editData(props.row)"
                size="sm"
                color="grey-6"
                round
                dense
                icon="edit"
              />
              <q-btn
                @click="confirmDelete(props.row.id)"
                size="sm"
                color="grey-6"
                round
                dense
                icon="delete"
              />
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

<script setup>
import axios from 'axios';
import { ref } from 'vue';
import { useQuasar } from 'quasar';

const $q = useQuasar();

const blockData = ref([]);
const tableConfig = [
  { label: '姓名', name: 'name', field: 'name', align: 'left' },
  { label: '年齡', name: 'age', field: 'age', align: 'left' },
];

const tempData = ref({ id: null, name: '', age: null });

async function fetchData() {
  try {
    const response = await axios.get(
      'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    );
    blockData.value = response.data;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

async function createOrUpdateData() {
  try {
    if (tempData.value.id) {
      await axios.patch(
        'https://dahua.metcfire.com.tw/api/CRUDTest',
        tempData.value
      );
      const index = blockData.value.findIndex(
        (item) => item.id === tempData.value.id
      );
      if (index !== -1) {
        blockData.value[index] = { ...tempData.value };
      }
    } else {
      const response = await axios.post(
        'https://dahua.metcfire.com.tw/api/CRUDTest',
        tempData.value
      );
      blockData.value.push(response.data);
    }
    tempData.value = { id: null, name: '', age: null };
  } catch (error) {
    console.error(
      'Error creating or updating data:',
      error.response ? error.response.data : error.message
    );
  }
}

async function deleteData(id) {
  try {
    await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);
    blockData.value = blockData.value.filter((item) => item.id !== id);
  } catch (error) {
    console.error('Error deleting data:', error);
  }
}

function editData(data) {
  tempData.value = { ...data };
}

function confirmDelete(id) {
  $q.dialog({
    title: '確認刪除',
    message: '你確定要刪除此條目嗎？',
    ok: {
      label: '確認',
      color: 'negative',
    },
    cancel: {
      label: '取消',
      color: 'primary',
    },
  })
    .onOk(() => {
      deleteData(id);
    })
    .onCancel(() => {
      console.log('取消刪除操作');
    });
}

fetchData();
</script>

<style scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}
.q-table tbody td {
  font-size: 18px;
}
tr th:nth-child(1) {
  width: 80%;
}
tr th:nth-child(2),
tr th:nth-child(3) {
  width: 10%;
}
tr button {
  margin-right: 1rem;
}
</style>
