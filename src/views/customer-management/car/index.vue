<template>
  <div class="p-2">
    <transition :enter-active-class="proxy?.animate.searchAnimate.enter" :leave-active-class="proxy?.animate.searchAnimate.leave">
      <div v-show="showSearch" class="mb-[10px]">
        <el-card shadow="hover">
          <el-form ref="queryFormRef" :model="data.queryParams" :inline="true" @submit.prevent>
            <el-form-item label="车辆品牌" prop="tagId">
              <el-select v-model="data.queryParams.brandId" placeholder="请选择车辆品牌" clearable filterable>
                <el-option v-for="item in []" :key="item.value" :label="item.label" :value="item.value"> </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="车辆厂商" prop="nickname">
              <el-select v-model="data.queryParams.manufacturer" placeholder="请选择车辆厂商" clearable filterable>
                <el-option v-for="item in []" :key="item.value" :label="item.label" :value="item.value"> </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="车辆系列" prop="telephone">
              <el-input v-model="data.queryParams.typename" placeholder="请输入车辆系列" clearable @keyup.enter="handleQuery" />
            </el-form-item>
            <el-form-item label="车辆型号" prop="label">
              <el-input v-model="data.queryParams.vehicleModel" placeholder="请输入车辆型号" clearable @keyup.enter="handleQuery" />
            </el-form-item>
            <el-form-item label="车架号码" prop="tagId">
              <el-input v-model="data.queryParams.vin" placeholder="请输入车架号码" clearable @keyup.enter="handleQuery" />
            </el-form-item>
            <el-form-item label="车辆号码" prop="nickname">
              <el-input v-model="data.queryParams.licensePlate" placeholder="请输入车辆号码" clearable @keyup.enter="handleQuery" />
            </el-form-item>
            <el-form-item label="客户昵称" prop="telephone"> </el-form-item>
            <el-form-item label="电话号码" prop="label"> </el-form-item>
            <el-form-item>
              <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
              <el-button icon="Refresh" @click="resetQuery">重置</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </div>
    </transition>
    <el-card shadow="hover">
      <template #header>
        <el-row :gutter="10" class="mb8">
          <el-col :span="1.5">
            <el-button v-hasPermi="['system:post:add']" type="primary" plain icon="Plus" @click="handleAdd">新增</el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button v-hasPermi="['system:post:remove']" type="danger" plain icon="Delete" :disabled="tableAttr.multiple" @click="handleDelete()">
              删除
            </el-button>
          </el-col>
          <right-toolbar v-model:showSearch="showSearch" @query-table="getTableData"></right-toolbar>
        </el-row>
      </template>
      <el-table v-loading="loading" :data="tableData" tooltip-effect="dark myTooltips" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55" align="center" />
        <el-table-column label="车辆品牌" align="center" prop="brandId" />
        <el-table-column label="车辆号码" align="center" prop="licensePlate" />
        <el-table-column label="车架号码" align="center" prop="vin" />
        <el-table-column label="车辆厂商" align="center" prop="manufacturer" />
        <el-table-column label="车辆系列" align="center" prop="typename" />
        <el-table-column label="车辆型号" align="center" prop="vehicleModel" />
        <el-table-column label="车辆状态" align="center" prop="" />
        <el-table-column label="客户昵称" align="center" prop="" />
        <el-table-column label="电话号码" align="center" prop="" />
        <el-table-column label="操作" width="220" align="center" class-name="small-padding fixed-width">
          <template #default="{ row }">
            <el-tooltip content="装配情况" placement="top">
              <el-button v-hasPermi="['system:post:edit']" link type="primary" icon="Edit" @click=""></el-button>
            </el-tooltip>
            <el-tooltip content="订单记录" placement="top">
              <el-button v-hasPermi="['system:post:edit']" link type="primary" icon="Edit" @click="handleChangePhone(row)"></el-button>
            </el-tooltip>
            <el-tooltip content="变更车主" placement="top">
              <el-button v-hasPermi="['system:post:edit']" link type="primary" icon="Edit" @click=""></el-button>
            </el-tooltip>
            <el-tooltip content="禁用、启用" placement="top">
              <el-button v-hasPermi="['system:post:edit']" link type="primary" icon="Edit" @click=""></el-button>
            </el-tooltip>
            <el-tooltip content="编辑" placement="top">
              <el-button v-hasPermi="['system:post:edit']" link type="primary" icon="Edit" @click="handleUpdate(row)"></el-button>
            </el-tooltip>
            <el-tooltip content="删除" placement="top">
              <el-button v-hasPermi="['system:post:remove']" link type="danger" icon="Delete" @click="handleDelete(row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <pagination
        v-show="tableAttr.total > 0"
        v-model:page="data.queryParams.pageNum"
        v-model:limit="data.queryParams.pageSize"
        :total="tableAttr.total"
        @pagination="getTableData"
      />
    </el-card>

    <!-- 添加或修改对话框 -->
    <el-dialog v-model="dialog.visible" :title="dialog.title" width="600px" append-to-body>
      <el-form ref="FormDataRef" :model="data.form" :rules="data.rules" label-width="80px" @submit.prevent>
        <el-form-item label="选择车主" prop="customId">
          <el-select v-model="data.form.customId" placeholder="请选择客户标签" clearable filterable>
            <el-option v-for="item in dictObj.carManageerTag" :key="item.value" :label="item.label" :value="item.value"> </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="车辆归属" prop="toType">
          <el-radio-group v-model="data.form.toType">
            <el-radio v-for="item in dictObj.belong" :key="item.key" :label="item.label" :value="item.value" />
          </el-radio-group>
        </el-form-item>
        <el-form-item label="车辆品牌" prop="brandId">
          <el-select v-model="data.form.brandId" placeholder="请选择渠道来源" clearable filterable>
            <el-option v-for="item in dictObj.carManage" :key="item.value" :label="item.label" :value="item.value"> </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="车牌情况" prop="licensePlateState">
          <el-radio-group v-model="data.form.licensePlateState">
            <el-radio v-for="item in dictObj.cardNoState" :key="item.key" :label="item.label" :value="item.value" />
          </el-radio-group>
        </el-form-item>
        <el-form-item label="车辆号牌" prop="licensePlate">
          <div class="cardNo">
            <el-select v-model="data.form.licenseProvince" placeholder="京" clearable filterable class="cardNo--province">
              <el-option v-for="item in dictObj.carProvince" :key="item.value" :label="item.label" :value="item.value"> </el-option>
            </el-select>
            <el-select v-model="data.form.licenseOrg" placeholder="A" clearable filterable class="cardNo--city">
              <el-option v-for="item in dictObj.carCity" :key="item.value" :label="item.label" :value="item.value"> </el-option>
            </el-select>
            <el-input v-model="data.form.licenseNum" row="auto" placeholder="请输入内容" class="cardNo--number" />
          </div>
        </el-form-item>
        <el-form-item label="车辆照片" prop="remarks">
          <image-upload v-model="data.form.remarks"></image-upload>
        </el-form-item>
        <el-form-item label="备注" prop="remarks">
          <el-input v-model="data.form.remarks" type="textarea" row="auto" placeholder="请输入内容" />
        </el-form-item>
        <el-form-item label="信息补全" prop="remarks">
          <el-radio-group v-model="data.form.remarks">
            <el-radio v-for="item in dictObj.infoComp" :key="item.key" :label="item.label" :value="item.value" />
          </el-radio-group>
        </el-form-item>

        <el-descriptions class="margin-top" title="车辆信息" :column="2" border>
          <el-descriptions-item label="车辆品牌"> {{}} </el-descriptions-item>
          <el-descriptions-item label="车辆厂商"> {{}} </el-descriptions-item>
          <el-descriptions-item label="车辆系列"> {{}} </el-descriptions-item>
          <el-descriptions-item label="车辆型号"> {{}} </el-descriptions-item>
          <el-descriptions-item label="车辆级别"> {{}} </el-descriptions-item>
          <el-descriptions-item label="车身结构"> {{}} </el-descriptions-item>
          <el-descriptions-item label="驱动方式"> {{}} </el-descriptions-item>
          <el-descriptions-item label="能源类型"> {{}} </el-descriptions-item>
        </el-descriptions>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>
<script setup name="carManage" lang="ts">
import { carManageList, carManageAdd, carManageDel, carManageInfo, carManageUp } from '@/api/customer-management/car';
import { FormData, TableQuery, TableVO } from '@/api/customer-management/car/types';
import { carProvince, carCity } from '@/utils/static-dict';

const { proxy } = getCurrentInstance() as ComponentInternalInstance;
const dictObj = toReactive<any>(proxy?.useDict('sys_user_sex', 'clyh_staff_entry_state'));
dictObj.carProvince = carProvince;
dictObj.carCity = carCity;
dictObj.belong = [
  { label: '个人', value: '0' },
  { label: '企业', value: '1' }
];
dictObj.cardNoState = [
  { label: '已上牌', value: '0' },
  { label: '随机车牌', value: '1' }
];
dictObj.infoComp = [
  { label: '稍后补全', value: '0' },
  { label: '立即补全', value: '1' }
];

const tableData = ref<TableVO[]>([]);
const loading = ref(true);
const showSearch = ref(true);
const tableAttr = reactive<TableAttr>({
  ids: [],
  multiple: true,
  total: 0
});

const queryFormRef = ref<ElFormInstance>();

const pageTitle = '渠道来源';
const FormDataRef = ref<ElFormInstance>();

const dialog = reactive<DialogOption>({
  visible: false,
  title: ''
});

const initFormData: FormData = {
  id: undefined,
  customId: '',
  toType: '',
  brandId: '',
  licensePlateState: '',
  licenseProvince: '',
  licenseOrg: '',
  licenseNum: '',
  licenseBit: '',
  licensePlate: '',
  imgUrls: '',
  remarks: '',
  // 信息补全
  carState: '',
  vin: '',
  manufacturer: '',
  typename: '',
  vehicleModel: '',
  sizetype: '',
  bodytype: '',
  drivemode: '',
  fueltype: ''
};

const data = reactive<PageData<FormData, TableQuery>>({
  form: { ...initFormData },
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    brandId: '',
    manufacturer: '',
    typename: '',
    vehicleModel: '',
    vin: '',
    licensePlate: ''
  },
  rules: {
    tagId: [{ required: true, message: '客户标签不能为空', trigger: ['blur', 'change'] }],
    nickname: [{ required: true, message: '客户昵称不能为空', trigger: 'blur' }],
    telephone: [{ required: true, message: '手机号码不能为空', trigger: 'blur' }],
    channel: [{ required: true, message: '渠道来源不能为空', trigger: ['blur', 'change'] }]
  }
});

/** 查询列表 */
const getTableData = async () => {
  loading.value = true;
  const res = await carManageList(data.queryParams);
  tableData.value = res.rows;
  tableAttr.total = res.total;
  loading.value = false;
};

/** 取消按钮 */
const cancel = () => {
  reset();
  dialog.visible = false;
};

/** 表单重置 */
const reset = () => {
  data.form = { ...initFormData };
  FormDataRef.value?.resetFields();
};

/** 搜索按钮操作 */
const handleQuery = () => {
  data.queryParams.pageNum = 1;

  getTableData();
};

/** 重置按钮操作 */
const resetQuery = () => {
  queryFormRef.value?.resetFields();
  data.queryParams.pageNum = 1;

  handleQuery();
};

/** 多选框选中数据 */
const handleSelectionChange = (selection: TableVO[]) => {
  tableAttr.ids = selection.map((item) => item.id);
  tableAttr.multiple = !selection.length;
};

/** 新增按钮操作 */
const handleAdd = () => {
  reset();
  dialog.visible = true;
  dialog.title = `添加${pageTitle}`;
};

/** 修改按钮操作 */
const handleUpdate = async (row?: TableVO) => {
  reset();
  const postId = row?.id || tableAttr.ids[0];
  const res = await carManageInfo(postId);
  Object.assign(data.form, res.data);
  dialog.visible = true;
  dialog.title = `修改${pageTitle}`;
};

/** 提交按钮 */
const submitForm = () => {
  FormDataRef.value?.validate(async (valid: boolean) => {
    if (valid) {
      data.form.id ? await carManageUp(data.form) : await carManageAdd(data.form);
      proxy?.$modal.msgSuccess('操作成功');
      dialog.visible = false;
      await getTableData();
    }
  });
};

/** 账户充值 */

/** 修改手机号 */
type ChangeDialog = {
  visible: boolean;
  form: PhoneForm;
};
const changeDialog = reactive<ChangeDialog>({
  visible: false,
  form: {
    id: '',
    carManageNo: '',
    tagId: '',
    nickname: '',
    telephone: ''
  }
});
const handleChangePhone = async (row?: TableVO) => {
  const postId = row?.id || tableAttr.ids[0];
  const res = await carManageInfo(postId);
  Object.assign(changeDialog.form, res.data);
  changeDialog.visible = true;
};

/** 客户档案 */

/** 删除按钮操作 */
const handleDelete = async (row?: TableVO) => {
  const ids = row?.id || tableAttr.ids;
  await proxy?.$modal.confirm('是否删除选中项？');
  await carManageDel(ids);
  await getTableData();
  proxy?.$modal.msgSuccess('删除成功');
};
const init = async () => {
  getTableData();
};

init();
</script>
<style lang="scss" scoped>
.cardNo {
  display: flex;
  &--city {
    width: 80px;
  }
  &--province {
    width: 80px;
  }
  &--number {
    flex: 1;
  }
}
</style>
