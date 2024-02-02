<template>
  <div>
    <avue-crud
      v-model="form"
      :option="option"
      :page="page"
      :data="dataList"
      :upload-after="handleUploadAfter"
      :upload-delete="handleUploadDelete"
      @on-load="getMatterList"
      @refresh-change="getMatterList"
      @row-del="onDelete"
      @row-update="handleRowUpdate"
      @row-save="handleRowSave"
      ref="crud"
    >
      <template 
        #menu="{row,index,size}"
      >
        <el-button 
          :size="size"
          type="primary"
          icon="el-icon-edit"
          link
          @click="handleRowEdit(row,index)">编辑</el-button>
        <el-button 
          :size="size"
          icon="el-icon-delete"
          type="primary"
          link
          @click="$refs.crud.rowDel(row,index)">删除</el-button>
      </template>
  </avue-crud>
  </div>
</template>
<script setup>
import cookie from 'vue-cookies'
import { ElMessage, ElMessageBox } from 'element-plus'

const form = ref()
const crud = ref(null);
const dataList = ref([])
const page = reactive({
  total: 0, // 总页数
  currentPage: 1, // 当前页数
  pageSize: 20 // 每页显示多少
})
const option = ref({
  menuHeaderAlign: 'center',
  menuAlign: 'center',
  headerAlign: 'center',
  align: 'center',
  editBtn: false,
  delBtn: false,
  column: [
    {
      label: 'Id',
      prop: 'id',
      width: 100,
      addDisplay: false,
      addDisabled: true,
      editDisabled: true,
      editDisplay: false,
    },
    {
      label: '标题',
      prop: 'title',
      rules: [
        {
          required: true,
          message: '请输入标题',
        }
      ],
    },
    {
      label: '内容',
      prop: 'content',
      overHidden: true,
      rules: [
        {
          required: true,
          message: '请输入内容',
        }
      ],
    },
    {
      label: '图片',
      prop: 'imageUrls',
      type: 'upload',
      headers: {
        'Content-Type': 'multipart/form-data',
        Authorization: cookie.get('Authorization')
      },
      propsHttp: {
        res: 'data'
      },
      span: 24,
      listType: 'picture-card',
      tip: '只能上传jpg/png文件，且不超过500kb',
      action: import.meta.env.VITE_APP_BASE_API + '/upload/img',
      hide: true,
    },
    {
      label: 'imgIdList',
      prop: 'imgIdList',
      type: 'array',
      addDisabled: true,
      editDisabled: true,
      hide: true,
      editDisplay: false,
      addDisplay: false,
    },

  ]
})
// 获取列表
const getMatterList = (pageParam, params, done) => {
  if (!pageParam) {
    pageParam = {
      currentPage: 1,
      pageSize: 20
    }
  }
  http({
    url: http.adornUrl('/matter/list'),
    method: 'get',
    params: http.adornParams({
      pageNo: pageParam.currentPage,
      pageSize: pageParam.pageSize,
    }),
  }).then((res) => {
    dataList.value = res.data.records;
    page.total = res.data.total;
    page.currentPage = res.data.current;
    if (done) done()
  })
}
// 删除
const onDelete = (form, index, done) => {
  ElMessageBox.confirm('此操作将永久删除该文件, 是否继续?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    http({
      url: http.adornUrl('/matter/del'),
      method: 'post',
      data: http.adornData({ id: form.id }),
    }).then((res) => {
      if (res.success) {
        if (done) done()
        ElMessage({
          message: '操作成功',
          type: 'success',
          duration: 1500,
        })
      }
    })
  })
}
// 上传后
const handleUploadAfter = (res, done) => {
  console.log(form.value.imgIdList)
  form.value.imgIdList = [...form.value.imgIdList, res.id];
  done()
}
const handleUploadDelete = async (file, column) => {
  const { imageUrls, imgIdList } = form.value;
  const index = imageUrls.find((item, index) => (item === file.url))
  const newImgIdList = imgIdList.toSpliced(index, 1)
  form.value.imgIdList = newImgIdList;
  return true;
}
const handleRowUpdate = (row, index, done, loading) => {
  loading();
  http({
    url: http.adornUrl('/matter/edit'),
    method: 'post',
    data: http.adornData(row),
  }).then((res) => {
    if (res.success) {
      ElMessage({
        message: '操作成功',
        type: 'success',
        duration: 1500,
      })
      getMatterList()
      done();
    }
  });
}
const handleRowSave = (row, done, loading) => {
  loading();
  http({
    url: http.adornUrl('/matter/add'),
    method: 'post',
    data: http.adornData(row),
  }).then((res) => {
    if (res.success) {
      ElMessage({
        message: '操作成功',
        type: 'success',
        duration: 1500,
      })
      done();
    }
  });
}
const handleRowEdit = (row, index) => {
  http({
    url: http.adornUrl('/matter/detail'),
    method: 'get',
    params: http.adornParams({ id: row.id }),
  }).then((res) => {
    // dataList.value[index] = res.data
    crud.value.rowEdit(res.data, index)
  });
}
</script>
<style lang="scss" scoped>
</style>
