<template>
  <div>
    <!-- LDAP Server Address Input and Confirm Button -->
    <el-row :gutter="20">
      <el-col :span="8">
        <el-input v-model="ldapServerAddress" placeholder="Enter LDAP Server Address"></el-input>
      </el-col>
      <el-col :span="8">
        <!-- Search Input for LDAP Objects -->
        <el-input v-model="filterText" placeholder="Search LDAP Objects"></el-input>
      </el-col>
      <el-col :span="8">
        <el-button type="primary" @click="confirmServerAddress">确定</el-button>
      </el-col>
    </el-row>


    <el-container>
      <!-- Left Side: LDAP Objects Tree -->
      <el-aside width="30%">
        <el-tree
            class="tree-wrapper"
            ref="treeRef"
            :data="ldapObjects"
            :props="defaultProps"
            :load="loadNode"
            lazy
            @node-click="showDetails"
            :highlight-current="true"
        >
          <template #default="{ node }">
            <span>{{ node.data.nodeName }}</span>
          </template>
        </el-tree>
      </el-aside>


      <!-- Right Side: LDAP Object Details Table -->
      <el-main>
        <!-- Breadcrumb for Navigation -->
        <el-breadcrumb>
          <el-breadcrumb-item v-for="item in breadcrumbItems" :key="item.label">{{ item.label }}</el-breadcrumb-item>
        </el-breadcrumb>
        <el-table :data="tableData" style="width: 100%">
          <el-table-column prop="attributeKey" label="Attribute"></el-table-column>
          <el-table-column prop="attributeValue" label="Value"></el-table-column>
        </el-table>
      </el-main>
    </el-container>
  </div>
</template>

<script lang="ts">
import {Ref, ref, watch, defineComponent} from 'vue';
import axios from 'axios';
import {ElTree} from "element-plus";
import type Node from 'element-plus/es/components/tree/src/model/node'

interface LdapNode {
  nodeName: string;
  nodeCode: string;
  children?: LdapNode[];
  attributes: Record<string, string>;
}

export default defineComponent({
  setup() {
    const ldapServerAddress: Ref<string> = ref('ldap://10.32.116.82:389');
    const searchQuery: Ref<string> = ref('');
    const ldapObjects: Ref<LdapNode[]> = ref([]);
    const tableData: Ref<Array<{ attributeKey: string, attributeValue: string }>> = ref([]);
    const breadcrumbItems: Ref<Array<{ label: string, value: string }>> = ref([]);
    const treeRef = ref<InstanceType<typeof ElTree> | null>(null);
    const filterText: Ref<string> = ref('');
    const defaultProps = {
      children: 'children',
      label: 'nodeName',
    };

    const confirmServerAddress = () => {
      if (filterText.value) {
        search(filterText.value);
      } else {
        fetchChildren(null);
      }
    };
    const search = async (filter: string) => {
      axios.post('/ldap/search', {url: ldapServerAddress.value, filter: filter}).then(response => {
        console.log(response.data);
        ldapObjects.value = response.data;

      }).catch(error => {
        console.error("Error fetching LDAP data:", error);
      });
    }
    const fetchChildren = async (node: Node | null) => {
      axios.post('/ldap/getSubNodeList', {url: ldapServerAddress.value, nodeCode: node?.data.nodeCode}).then(response => {
        if (node) {
          node.data.children = response.data;
        } else {
          ldapObjects.value = response.data;
        }
      }).catch(error => {
        console.error("Error fetching LDAP data:", error);
      });
    }

    const showDetails = async (nodeData: LdapNode) => {
      tableData.value = [];
      if ((nodeData.attributes as any) === null) {
        await axios.post('/ldap/getNode', {url: ldapServerAddress.value, nodeCode: nodeData.nodeCode}).then(response => {
          nodeData.attributes = response.data.attributes;
          console.log(nodeData.attributes, response.data.attributes);
        })
      }
      tableData.value = Object.entries(nodeData.attributes).map(([key, value]) => ({
            attributeKey: key,
            attributeValue: value
          }));
      updateBreadcrumb(nodeData);
    };

    const updateBreadcrumb = (nodeData: LdapNode) => {
      breadcrumbItems.value = nodeData.nodeCode.split(',').reverse().map(part => {
        const [label, value] = part.split('=');
        return {label: value, value};
      });
    };

    const loadNode = (node: Node, resolve: (data: LdapNode[]) => void) => {
      fetchChildren(node);
      if (node.level === 0) {
        resolve(ldapObjects.value);
      } else {
        resolve(node.data.children || []);
      }
    };

    return {
      ldapServerAddress,
      searchQuery,
      ldapObjects,
      breadcrumbItems,
      defaultProps,
      confirmServerAddress,
      showDetails,
      updateBreadcrumb,
      loadNode,
      treeRef,
      filterText,
      tableData
    };
  }
});

</script>

<style>
.tree-wrapper {
  max-height: calc(100vh - 50px); /* Adjust this value based on your needs */
  overflow-y: auto;
}
</style>