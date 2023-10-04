<template>
  <el-card>
    <el-table :data="ldapStatuses">
      <el-table-column prop="server" label="Server"></el-table-column>
      <el-table-column prop="isHealthy" label="Status" :formatter="formatStatus">
        <template slot-scope="scope">
          <el-tag :type="scope.row.isHealthy ? 'success' : 'danger'">
            {{ scope.row.isHealthy ? 'Healthy' : 'Unhealthy' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="responseTime" label="Response Time (ms)"></el-table-column>
      <el-table-column prop="totalConnections" label="Total Connections"></el-table-column>
    </el-table>
  </el-card>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      ldapStatuses: []
    };
  },
  methods: {
    formatStatus(row) {
      return row.isHealthy ? 'Healthy' : 'Unhealthy';
    },
    fetchStatus() {
      axios.get('/ldap/status').then(response => {
        this.ldapStatuses = Object.keys(response.data).map(server => {
          return {
            server,
            ...response.data[server]
          };
        });
      });
    }
  },
  mounted() {
    this.fetchStatus();
    setInterval(this.fetchStatus, 10000);
  }
};
</script>
