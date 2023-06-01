<template>
  <a-layout>
    <a-layout-content
        :style="{ background: '#fff', padding: '24px', margin: 0, minHeight: '280px' }"
    >
      <a-table
          :columns="columns"
          :row-key="record => record.id"
          :data-source="ebookList"
          :pagination="pagination"
          :loading="loading"
          @change="handleTableChange"
      >
        <template #cover="{ text : cover }">
          <img v-if="cover" style="height: 50px;width: 50px "  :src="cover" alt="avatar"/>
        </template>
        <template v-slot:action="{ text,record }">
          <a-space size="small">
            <a-button @click="showEditModal" type="primary">
              编辑
            </a-button>
            <a-button type="danger">
              删除
            </a-button>
          </a-space>
        </template>
      </a-table>
    </a-layout-content>
  </a-layout>

  <div>
    <a-modal
        title="电子书表单"
        v-model:visible="modalVisible"
        :confirm-loading="modalConfirmLoading"
        @ok="handleModalOk"
    >
      <p>modalText</p>
    </a-modal>
  </div>
</template>

<script lang="ts">
import {defineComponent, onMounted, ref} from 'vue';
import axios from "axios";

export default defineComponent({
  name: 'AdminEbook',
  setup(){
    const ebookList = ref();
    const pagination = ref({
      current: 1,
      pageSize: 4,
      total: 0
    });
    const loading = ref(false);
    const columns = [
      {
        title: '封面',
        dataIndex: 'cover',
        slots: {customRender: 'cover'}
      },
      {
        title: '名称',
        dataIndex: 'name'
      },
      {
        title: '分类一',
        key: 'category1Id',
        slots: {customRender: 'name'}
      },
      {
        title: '分类二',
        key: 'category2Id',
        slots: {customRender: 'name'}
      },
      {
        title: '文档数',
        dataIndex: 'docCount'
      },
      {
        title: '阅读数',
        dataIndex: 'viewCount'
      },
      {
        title: '点赞数',
        dataIndex: 'voteCount'
      },
      {
        title: 'Action',
        key: 'action',
        slots: {customRender: 'action'}
      }
    ];
    const modalVisible = ref<boolean>(false);
    const modalConfirmLoading = ref<boolean>(false);
    const showEditModal = () => {
      modalVisible.value = true;
    };
    const handleModalOk = () => {
      modalConfirmLoading.value = true;
      setTimeout(() => {
        modalVisible.value = false;
        modalConfirmLoading.value = false;
      }, 2000);
    };
    /**
     * 数据查询
     */
    const handleQuery = (params: any) => {
      loading.value = true;
      // console.log('params',params)
      axios.get("/ebook/getBookList",{
        params: params
      }).then((rsp)=>{
        loading.value = false;
        const data = rsp.data;
        ebookList.value = data.content.list;
      //   重置分页按钮
        pagination.value.current = params.page;
        pagination.value.total = data.content.total;
      })

    };
    /**
     * 表格点击页码时触发
     * @param pagination
     */
    const handleTableChange = (pagination: any) => {
      handleQuery({
        page: pagination.current,
        size: pagination.pageSize
      })
    };
    onMounted(()=>{
      handleQuery({
        page: 1,
        size: pagination.value.pageSize
      });
    })


    return {
      ebookList,
      pagination,
      loading,
      columns,
      handleTableChange,
      modalVisible,
      modalConfirmLoading,
      showEditModal,
      handleModalOk,
    }
  }

})
</script>

