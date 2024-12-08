<template>
  <div id="questionSubmitView">
    <a-form :model="searchParams" layout="inline">
      <a-form-item field="questionId" label="题号" tooltip="请输入题目Id">
        <a-input
          v-model="searchParams.questionId"
          placeholder="请输入搜索题号"
        />
      </a-form-item>
      <a-form-item
        field="submitLanguage"
        label="编程语言："
        style="min-width: 240px"
      >
        <a-select v-model="searchParams.language" placeholder="选择编程语言">
          <a-option>java</a-option>
          <a-option>cpp</a-option>
          <a-option>go</a-option>
        </a-select>
      </a-form-item>
      <a-form-item>
        <a-button type="outline" shape="round" status="normal" @click="doSubmit"
          >搜索
        </a-button>
      </a-form-item>
      <a-form-item>
        <a-button
          type="primary"
          shape="round"
          status="success"
          @click="loadData"
          >刷新
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
      <!-- 判题结果 -->
      <template #judgeResult="{ record }">
        <a-tag
          :color="record.judgeInfo?.message === 'Accepted' ? 'green' : 'red'"
        >
          {{ record.judgeInfo?.message || "未知" }}
        </a-tag>
      </template>
      <!-- 内存消耗 -->
      <template #judgeMemory="{ record }">
        {{ record.judgeInfo?.memory || 0 }} KB
      </template>
      <!-- 耗时 -->
      <template #judgeTime="{ record }">
        {{ record.judgeInfo?.time || 0 }} ms
      </template>
      <template #createTime="{ record }">
        {{ moment(record.createTime).format("YYYY-MM-DD HH:mm:ss") }}
      </template>
      <template #questionId="{ record }">
        <a-link
          status="success"
          style="color: blue"
          @click="toQuestionPage(record)"
          >{{ record.questionId }}
        </a-link>
      </template>
      <template #username="{ record }">
        {{ record.userVO?.userName || "未知用户" }}
      </template>
      <template #status="{ record }">
        <!--        判题状态（0 - 待判题、1 - 判题中、2 - 成功、3 - 失败）-->
        <a-tag v-if="record.status === 0" color="cyan">待判题</a-tag>
        <a-tag v-if="record.status === 1" color="green">判题中</a-tag>
        <a-tag v-if="record.status === 2" color="blue">成功</a-tag>
        <a-tag v-if="record.status === 3" color="red">失败</a-tag>
      </template>
    </a-table>
  </div>
</template>
<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import {
  Question,
  QuestionControllerService,
  QuestionSubmitQueryRequest,
  QuestionSubmitVO,
} from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";
import moment from "moment";

const tableRef = ref();
const dataList = ref([]);
const total = ref(0);
const searchParams = ref<QuestionSubmitQueryRequest>({
  questionId: undefined,
  language: undefined,
  pageSize: 10,
  current: 1,
});
const colors = ["orange", "green", "blue", "red"];
const loadData = async () => {
  const res = await QuestionControllerService.listQuestionSubmitByPageUsingPost(
    {
      ...searchParams.value,
      sortField: "createTime",
      sortOrder: "descend",
    }
  );
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
    title: "提交号",
    dataIndex: "id",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "提交者",
    slotName: "username",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "题号",
    slotName: "questionId",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "判题结果",
    slotName: "judgeResult",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "内存 (KB)",
    slotName: "judgeMemory",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "时间 (ms)",
    slotName: "judgeTime",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "编程语言",
    dataIndex: "language",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "判题状态",
    slotName: "status",
    align: "center", // 标题居中对齐
    titleAlign: "left", // 数据对齐方式
  },
  {
    title: "创建时间",
    slotName: "createTime",
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
const toQuestionPage = (questionSubmit: QuestionSubmitVO) => {
  router.push({
    path: `/view/question/${questionSubmit.questionId}`,
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
#questionSubmitView {
  max-width: 1280px;
  margin: 0 auto;
  border-radius: 10px;
}
</style>
