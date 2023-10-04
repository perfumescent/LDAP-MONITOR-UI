<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="8" v-for="(value, key) in ldapMetrics" :key="key">
        <el-card>
          <div slot="header" class="clearfix">
            <span>{{ formatHeader(key) }}</span>
          </div>
          <div class="text item">
            {{ value }}
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'LdapMetricsComponent',
  data() {
    return {
      ldapMetrics: {}
    };
  },
  methods: {
    formatHeader(key) {
      return key.replace(/([A-Z])/g, ' $1').replace(/^./, str => str.toUpperCase());
    },
    fetchMetrics() {
      axios.get('/ldap/metrics')
          .then(response => {
            if (typeof response.data === 'string') {
              // If the response is a string, try to parse it as JSON
              this.ldapMetrics = JSON.parse(response.data);
            } else {
              this.ldapMetrics = response.data;
            }
          })
          .catch(error => {
            console.error("Error fetching LDAP metrics:", error);
          });
    }
  },
  mounted() {
    this.fetchMetrics();
  }
}
</script>

<style scoped>
.text.item {
  font-size: 24px;
  text-align: center;
  margin-top: 20px;
}
</style>
