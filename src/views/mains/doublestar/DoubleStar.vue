<!--
    * @FileDescription: 双创之星页面。
    * @Author: 李雪茹
    * @Date: 2024年1月22日
    * @LastEditors: 李思佳
    * @LastEditTime: 2024年1月22日
-->
<template>
  <div>
    <a-spin tip="正在加载，请稍候..." :spinning="spinning">
      <a-table :columns="columns" :data-source="filteredData" :pagination="false" bordered :scroll="{x: 1300}">
        <template #title>
          <div>
            <a-input-search v-model:value="inputValue" placeholder="请输入内容进行搜索" class="search-input" @search="companysearch" size="large" />
            <a-button :icon="h(CloudDownloadOutlined)" type="primary" shape="round" @click="exportlistShuangc" class="right-button" size="large">导出双创之星名单</a-button>
          </div>
        </template>
        <!-- <a-input-search v-model:value="inputValue" placeholder="请输入内容" style="width: 200px" @search="companysearch" /> -->
        <template #bodyCell="{column, record}">
          <template v-if="column.dataIndex === 'url'">
            <a-button type="link" @click="showUrl(record)">点击查看</a-button>
          </template>
          <template v-else-if="column.dataIndex === 'state'">
            <div v-for="tag in record.state" :key="tag" :style="{color: tag === '1' ? 'green' : tag === '2' ? 'red' : 'black'}">
              {{ tag === '1' ? '已通过' : tag === '2' ? '未通过' : '未审核' }}
            </div>
          </template>
          <template v-else-if="column.dataIndex === 'operate'">
            <template v-for="action in record.state" :key="action">
              <!-- 状态为0 —— 未审核 -->
              <div v-if="action === '0'">
                <!-- 通过 -->
                <a-button ghost type="primary" @click="() => accept(record.id, record.url)" style="border-color: green; border-radius: 25px; color: #fff; background-color: green">
                  通 过
                </a-button>
                <!-- 未通过 -->
                <a-button ghost type="primary" @click="showReason(record.id)" danger style="border-radius: 25px; margin-left: 3%"> 未 通 过 </a-button>
              </div>
              <!-- 状态为1 —— 已审核通过 -->
              <div v-else-if="action === '1'">
                <!-- <a-button type="text" style="color: green" @click="showCertificate">查看证书</a-button> -->
                <CheckCircleTwoTone :style="{fontSize: '25px'}" />
              </div>
              <!-- 状态为2 —— 已审核不通过 -->
              <div v-else-if="action === '2'">
                <a-button type="dashed" @click="() => getRefusereason(record.id)">查看驳回理由</a-button>
              </div>
            </template>
          </template>
        </template>
      </a-table>
    </a-spin>
    <a-modal v-model:open="openReason" @ok="() => refuse(recordId, reasonValue)" @cancel="handleCancel" title="请输入不通过理由" ok-text="确定" cancel-text="取消" centered>
      <a-textarea v-model:value="reasonValue" :row="4"></a-textarea>
    </a-modal>
    <a-modal v-model:open="openRefusereason" title="查看详情" :ok-button-props="{disabled: true}" :cancel-button-props="{disabled: true}" centered :footer="null">
      <p>{{ checkReason }}</p>
    </a-modal>
  </div>
</template>

<script setup lang="ts">
import {ref, h} from 'vue';
import {CheckCircleTwoTone, CloudDownloadOutlined} from '@ant-design/icons-vue';
import {message} from 'ant-design-vue';
import {LXRgetcompany, LXRcompanysearch, LXRacceptShuangc, LXRrefuseShuangc, LXRgetreasonShuangc} from '@/service/main/double-star';
const inputValue = ref<string>('');
// 定义加载状态
const spinning = ref<boolean>(true);
const columns = [
  {
    title: '年级',
    dataIndex: 'grade',
    align: 'center',
    key: 'grade'
  },
  {
    title: '专业',
    dataIndex: 'major',
    align: 'center',
    key: 'major'
  },
  {
    title: '班级',
    dataIndex: 'class',
    align: 'center',
    key: 'class'
  },
  {
    title: '姓名',
    dataIndex: 'stuname',
    align: 'center',
    key: 'stuname'
  },
  {
    title: '注册公司名称',
    dataIndex: 'companyname',
    align: 'center',
    key: 'companyname'
  },
  {
    title: '实体/虚体',
    dataIndex: 'vp',
    align: 'center',
    key: 'vp'
  },
  {
    title: '申报人排名',
    dataIndex: 'ranking',
    align: 'center',
    key: 'ranking'
  },
  {
    title: '注册时间',
    dataIndex: 'signuptime',
    align: 'center',
    key: 'signuptime'
  },
  {
    title: '公司规模',
    dataIndex: 'scale',
    align: 'center',
    key: 'scale'
  },
  {
    title: '佐证材料',
    dataIndex: 'url',
    align: 'center',
    key: 'url'
  },
  {
    title: '状态',
    dataIndex: 'state',
    align: 'center',
    key: 'state'
  },
  {
    title: '操作',
    dataIndex: 'operate',
    align: 'center',
    fixed: 'right',
    width: 200,
    key: 'operate'
  }
];
/**
 * @description 佐证材料相关。
 */
const showUrl = (record) => {
  window.open(record.url, '_blank');
};
const filteredData = ref([]);
/**
 * @description 请求双创之星数据。
 */
const getData = async () => {
  const loginResult = await LXRgetcompany();
  // console.log(loginResult);
  if (loginResult.code) {
    filteredData.value = loginResult.data;
    spinning.value = false;
  } else {
    message.warning(`${loginResult.msg}`);
  }
};
getData();
/**
 * @description 双创之星搜索
 */
const companysearch = async () => {
  if (inputValue.value) {
    spinning.value = true;
    const searchResult = await LXRcompanysearch(inputValue.value);
    // console.log(searchResult);
    if (searchResult.code === 200) {
      spinning.value = false;
      filteredData.value = searchResult.data;
    } else {
      message.warning(`${searchResult.message}`);
    }
  } else {
    getData();
  }
};
/**
 * @description 导出双创之星的名单
 */
const exportlistShuangc = () => {
  window.open('http://47.108.144.113:2000/api/admin/exportlistShuangc?token=' + localStorage.getItem('LOGIN_TOKEN'));
};
/**
 * @description 审批通过双创。
 */
const accept = async (form_id, awardurl) => {
  spinning.value = true;

  const acceptResult = await LXRacceptShuangc(form_id, awardurl);
  spinning.value = false;

  // console.log(acceptResult);
  if (acceptResult.code) {
    // loading.value = false;
    message.success(`${acceptResult.msg}`);
    getData();
  } else {
    message.warning(`${acceptResult.msg}`);
  }
};
/**
 *@description 审批拒绝双创相关
 */
const recordId = ref('');

const openReason = ref<boolean>(false);
const reasonValue = ref('');
const showReason = (form_id) => {
  openReason.value = true;
  recordId.value = form_id;
};
const handleCancel = () => {
  openReason.value = false;
  reasonValue.value = '';
};
/**
 *@description 审批拒绝双创。
 */
const refuse = async (form_id, reason) => {
  if (reasonValue.value) {
    openReason.value = false;
    spinning.value = true;
    const refuseResult = await LXRrefuseShuangc(form_id, reason);
    // console.log(refuseResult);
    if (refuseResult.code) {
      refuseResult.data = reasonValue.value;
      getData();
      spinning.value = false;
      message.success(`${refuseResult.msg}`);
    } else {
      spinning.value = false;
      message.warning(`${refuseResult.msg}`);
    }
  } else {
    message.warning('拒绝理由不能为空！');
  }
};
/**
 *@description 查看驳回理由相关
 * */
let checkReason = ref('');
const openRefusereason = ref<boolean>(false);
const getRefusereason = async (form_id) => {
  // console.log('form_id:', form_id);
  spinning.value = true;
  const reasonResult = await LXRgetreasonShuangc(form_id);
  // console.log(reasonResult);
  if (reasonResult.code) {
    openRefusereason.value = true;
    // 存储驳回理由数据
    checkReason.value = reasonResult.data[0].reason;
    spinning.value = false;
  } else {
    spinning.value = false;
    message.warning(`${reasonResult.msg}`);
  }
};
/**
 * @description 查看证书相关。
 */
// const openCertificate = ref<boolean>(false);
// const showCertificate = () => {
//   openCertificate.value = true;
// };
// const handleMenuClick: MenuProps['onClick'] = (e) => {
//   console.log('click', e);
// };
</script>

<style scoped>
.search-input {
  width: 30%;
}
.right-button {
  float: right;
  width: 30%;
}
.top {
  display: flex;
  justify-content: space-between;
  margin-bottom: 30px;
}
@media (max-width: 768px) {
  .top {
    flex-direction: column;
  }
  .right {
    width: 200px;
    margin-top: 5px;
  }
  .responsive-table {
    max-width: 100%;
    overflow-x: auto;
  }
}
</style>
