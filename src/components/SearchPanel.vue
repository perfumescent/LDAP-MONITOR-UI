<template>
  <div>
    <!-- Input for LDAP Search -->
    <el-input v-model="url" placeholder="LDAP URL"></el-input>
    <el-input v-model="filter" placeholder="Filter"></el-input>
    <el-button @click="searchLDAP">Search</el-button>

    <el-container>
      <!-- Left Side: LDAP Objects Tree -->
      <el-aside width="30%">
        <el-tree :data="ldapObjects" node-key="uid" @node-click="showDetails">
          <template #default="{ node }">
            <span>{{ node.data.cn?"cn="+node.data.cn:"ou="+node.data.ou}}</span>
          </template>
        </el-tree>
      </el-aside>

      <!-- Right Side: LDAP Object Details Table -->
      <el-main>
        <el-table :data="selectedObjectDetails" style="width: 100%">
          <el-table-column prop="key" label="Attribute"></el-table-column>
          <el-table-column prop="value" label="Value"></el-table-column>
        </el-table>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import { ref } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const url = ref('');
    const filter = ref('');
    const ldapObjects = ref([]);
    const selectedObjectDetails = ref([]);

    const searchLDAP = async () => {
      try {
        const response = await axios.post('/ldap/search', { url: url.value, filter: filter.value });
        ldapObjects.value = response.data;
      } catch (error) {
        console.error("Error searching LDAP:", error);
      }
    };

    const showDetails = (nodeData) => {
      selectedObjectDetails.value = Object.entries(nodeData).map(([key, value]) => ({ key, value }));
    };

    return {
      url,
      filter,
      ldapObjects,
      selectedObjectDetails,
      searchLDAP,
      showDetails
    };
  }
};
</script>
