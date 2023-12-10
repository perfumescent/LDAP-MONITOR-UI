<template>

  <div>
    <el-button @click="showAddModal = true" type="primary">新增配置</el-button>
    <el-dialog title="新增LDAP配置" v-model="showAddModal" width="30%">
      <el-form :model="newConfig" label-width="120px" label-position="top">
        <el-form-item label="服务器名称">
          <el-input v-model="newConfig.serverName"/>
        </el-form-item>
        <el-form-item label="服务器URL">
          <el-input v-model="newConfig.serverUrl"/>
        </el-form-item>
        <el-form-item label="Base DN">
          <el-input v-model="newConfig.baseDn"/>
        </el-form-item>
        <el-form-item label="用户DN">
          <el-input v-model="newConfig.userDn"/>
        </el-form-item>
        <el-form-item label="密码">
          <el-input type="password" v-model="newConfig.password"/>
        </el-form-item>
        <el-form-item label="描述">
          <el-input type="textarea" v-model="newConfig.description"/>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="showAddModal = false">取消</el-button>
    <el-button type="primary" @click="addConfig">保存</el-button>
  </span>
    </el-dialog>


    <el-table :data="configs" style="width: 100%">
      <el-table-column prop="serverName" label="服务器名称"></el-table-column>
      <el-table-column prop="serverUrl" label="服务器URL"></el-table-column>
      <el-table-column prop="baseDn" label="Base DN"></el-table-column>
      <el-table-column prop="userDn" label="用户DN"></el-table-column>
      <el-table-column prop="password" label="密码"></el-table-column>
      <el-table-column prop="description" label="描述"></el-table-column>
      <el-table-column label="操作">
        <template #default="{ row }">
          <el-button @click="setCurrentConfig(row)">编辑</el-button>
          <el-button @click="deleteConfig(row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog
        title="当前配置详情"
        v-model="dialogVisible"
        width="50%">
      <el-form :model="currentConfig">
        <el-form-item label="服务器名称">
          <el-input v-model="currentConfig.serverName"/>
        </el-form-item>
        <el-form-item label="服务器URL">
          <el-input v-model="currentConfig.serverUrl"/>
        </el-form-item>
        <el-form-item label="Base DN">
          <el-input v-model="currentConfig.baseDn"/>
        </el-form-item>
        <el-form-item label="用户DN">
          <el-input v-model="currentConfig.userDn"/>
        </el-form-item>
        <el-form-item label="密码">
          <el-input v-model="currentConfig.password"/>
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="currentConfig.description"/>
        </el-form-item>
        <el-form-item>
          <el-button @click="updateConfig">更新</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import {ref} from 'vue';
import axios from 'axios';

interface LdapServerConfig {
  id: number;
  serverName: string;
  serverUrl: string;
  baseDn: string;
  userDn: string;
  password: string;
  description: string;
}

const configs = ref<LdapServerConfig[]>([]);
const currentConfig = ref<LdapServerConfig | null>(null);
const showAddModal = ref(false);
const newConfig = ref<LdapServerConfig>({
  id: -1,
  serverName: '',
  serverUrl: '',
  baseDn: '',
  userDn: '',
  password: '',
  description: ''
});
const dialogVisible = ref(false)
const addConfig = async () => {
  try {
    await axios.post('/ldap/config/create', newConfig.value);
    fetchConfigs();
    showAddModal.value = false;
  } catch (error) {
    console.error('Failed to add config:', error);
  }
};
const fetchConfigs = async () => {
  try {
    const response = await axios.get('/ldap/config/get');
    configs.value = response.data;
  } catch (error) {
    console.error('Failed to fetch configs:', error);
  }
};

const setCurrentConfig = (config: LdapServerConfig) => {
  currentConfig.value = {...config};
  dialogVisible.value = true;
};

const updateConfig = async () => {
  if (!currentConfig.value) return;
  try {
    await axios.post('/ldap/config/update', currentConfig.value);
    fetchConfigs();
    dialogVisible.value = false;
  } catch (error) {
    console.error('Failed to update config:', error);
  }
};

const deleteConfig = async (id: string) => {
  try {
    await axios.get(`/ldap/config/delete?id=${id}`);
    fetchConfigs();
  } catch (error) {
    console.error('Failed to delete config:', error);
  }
};

fetchConfigs();
</script>
