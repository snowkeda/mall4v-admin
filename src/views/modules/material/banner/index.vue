<template>
  <div>
    <avue-crud
      v-model="form"
      :option="option"
      :data="dataList"
      :upload-after="handleUploadAfter"
      @on-load="getBannerList"
      @row-del="onDelete"
      @row-save="handleRowSave"
    >
      <!-- <template #menu-left>
        <el-button
          v-if="isAuth('admin:indexImg:save')"
          type="primary"
          icon="el-icon-plus"
          @click.stop="onAddOrUpdate()"
        >
          新增
        </el-button>
      </template> -->
    </avue-crud>
  </div>
</template>
<script setup>
import cookie from 'vue-cookies'
import { ElMessage, ElMessageBox } from 'element-plus'

const option = ref({
  menuHeaderAlign: 'center',
  menuAlign: 'center',
  headerAlign: 'center',
  align: 'center',
  editBtn: false,
  column: [
    {
      label: 'Id',
      prop: 'id',
      width: 100,
      addDisabled: true,
      editDisabled: true,
    },
    {
      label: '图片',
      prop: 'bannerUrl',
      type: 'upload',
      listType: 'picture-img',
      span: 24,
      headers: {
        'Content-Type': 'multipart/form-data',
        Authorization: cookie.get('Authorization')
      },
      propsHttp: {
        res: 'data'
      },
      rules: [
        {
          required: true,
          message: '请选择上传的图片',
        }
      ],
      tip: '只能上传jpg/png用户头像，且不超过500kb',
      action: import.meta.env.VITE_APP_BASE_API + '/upload/img',
    },
  ]
});
const form = ref()
const dataList = ref([])
const handleUploadAfter = (res, done) => {
  form.value.id = res.id;
  done()
}
const handleRowSave = (row, done, loading) => {
  loading();
  http({
    url: http.adornUrl('/matter/add-banner'),
    method: 'post',
    data: http.adornData({ id: row.id }),
  }).then((res) => {
    getBannerList()
    done()
  })
}
const getBannerList = (pageParam, params, done) => {
  http({
    url: http.adornUrl('/matter/banner-list'),
    method: 'get',
  }).then((res) => {
    dataList.value = res.data;
    if (done) done()
  })
}
const onDelete = (form, index, done) => {
  // console.log(form.id)
  ElMessageBox.confirm('此操作将永久删除该文件, 是否继续?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    http({
      url: http.adornUrl('/matter/del-banner'),
      method: 'post',
      data: http.adornData({ id: form.id }),
    }).then((res) => {
      if (done) done()
      ElMessage({
        message: '操作成功',
        type: 'success',
        duration: 1500,
        onClose: () => {
          getBannerList()
        }
      })
    })
  })
}
</script>
<style lang="scss" scoped>
</style>
