<template>
  <div id="questionsView">
    <a-form :model="searchParams" layout="inline">
      <a-form-item
        field="title"
        label="题目："
        tooltip="请输入搜索的题目"
        style="min-width: 240px"
      >
        <a-input v-model="searchParams.title" placeholder="请输入搜索题目" />
      </a-form-item>
      <a-form-item
        field="tags"
        label="题目标签："
        tooltip="请输入搜索题目标签"
        style="min-width: 240px"
      >
        <a-input-tag v-model="searchParams.tags" placeholder="请输入题目标签" />
      </a-form-item>
      <a-form-item>
        <a-button type="outline" shape="round" status="normal" @click="doSubmit"
          >搜索
        </a-button>
      </a-form-item>
    </a-form>
    <a-divider size="0"></a-divider>
    <a-table
      :ref="tableRef"
      :columns="columns"
      :data="dataList"
      :pagination="{
        showTotal: true,
        pageSize: searchParams.pageSize,
        current: searchParams.current,
        total,
        showJumper: true,
        showPageSize: true,
      }"
      :scroll="{ x: 'max-content' }"
      @page-change="onPageChange"
      @pageSizeChange="onPageSizeChange"
    >
      <template #tags="{ record }">
        <a-space wrap>
          <a-tag v-for="(tag, index) of record.tags" :key="index" color="green"
            >{{ tag }}
          </a-tag>
        </a-space>
      </template>
      <template #acceptedRate="{ record }">
        {{
          `${
            Math.round(
              (record.submitNum > 0
                ? (record.acceptedNum / record.submitNum) * 100
                : "0" * 100) * 100
            ) / 100
          }% (${record.acceptedNum}/${record.submitNum})`
        }}
      </template>
      <template #createTime="{ record }">
        {{ moment(record.createTime).format("YYYY-MM-DD HH:mm:ss") }}
      </template>
      <template #optional="{ record }">
        <a-space>
          <a-button
            shape="round"
            status="normal"
            type="primary"
            @click="toQuestionPage(record)"
          >
            做题
          </a-button>
        </a-space>
      </template>
    </a-table>
  </div>
</template>
<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import {
  Question,
  QuestionControllerService,
  QuestionQueryRequest,
} from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";
import moment from "moment";

const tableRef = ref();
const dataList = ref([]);
const total = ref(0);
const searchParams = ref<QuestionQueryRequest>({
  title: "",
  tags: [],
  pageSize: 10,
  current: 1,
});
const loadData = async () => {
  const res = await QuestionControllerService.listQuestionVoByPageUsingPost({
    ...searchParams.value,
    sortField: "createTime",
    sortOrder: "descend",
  });
  if (res.code === 0) {
    dataList.value = res.data.records;
    total.value = res.data.total;
  } else {
    message.error("加载失败" + res.message);
  }
};
/**
 * 监听 searchParams 变量，改变时触发页面的重新加载
 */
watchEffect(() => {
  loadData();
});

/**
 * 页面加载时，请求数据
 */
onMounted(() => {
  loadData();
});
const columns = [
  {
    title: "题号",
    dataIndex: "id",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "题目",
    dataIndex: "title",
  },
  {
    title: "标签",
    slotName: "tags",
  },
  {
    title: "通过率",
    slotName: "acceptedRate",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "创建时间",
    slotName: "createTime",
    sortable: {
      sortDirections: ["ascend"],
    },
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "操作",
    slotName: "optional",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
];

const onPageChange = (page: number) => {
  searchParams.value = {
    ...searchParams.value,
    current: page,
  };
};
const onPageSizeChange = (size: number) => {
  searchParams.value = {
    ...searchParams.value,
    pageSize: size,
  };
};
const router = useRouter();
//跳转到做题页面
const toQuestionPage = (question: Question) => {
  router.push({
    path: `/view/question/${question.id}`,
  });
};
const doSubmit = () => {
  //这里需要重置搜索页号
  searchParams.value = {
    ...searchParams.value,
    current: 1,
  };
};
</script>
<style scoped>
#questionsView {
  max-width: 1280px;
  margin: 0 auto;
  border-radius: 10px;
}
</style>
