<template>
  <el-dialog title="医保支付" :visible.sync="visible" width="600px" @close="handleClose">
    <div v-if="loading">正在加载医保信息...</div>
    <div v-else-if="psnInfo">
      <div>
        <p>姓名：{{ psnInfo.psn_name }}</p>
        <p>医保类型：{{ psnInfo.insutype }}</p>
        <p>医保账户余额：{{ psnInfo.balc }}</p>
      </div>

      <el-button type="primary" @click="preSettle">医保预结算</el-button>

      <div v-if="preSettleData">
        <p>医保可报销：{{ preSettleData.fund_pay_sumamt }}</p>
        <p>个人需支付：{{ preSettleData.psn_part_amt }}</p>
        <p>医保账户支付：{{ preSettleData.acct_pay }}</p>
        <p>医保账户余额（预结算后）：{{ preSettleData.balc }}</p>

        <el-button type="success" @click="doSettle">医保结算</el-button>

        <div v-if="needOtherPay">
          <p>医保余额不足，选择支付方式：</p>
          <el-radio-group v-model="payMethod">
            <el-radio label="支付宝"></el-radio>
            <el-radio label="微信"></el-radio>
            <el-radio label="现金"></el-radio>
          </el-radio-group>
          <el-button type="primary" @click="confirmOtherPay">确认支付</el-button>
        </div>
      </div>
    </div>
  </el-dialog>
</template>

<script>
import axios from 'axios'
import { Message } from 'element-ui'

export default {
  name: 'MiDialog',
  props: {
    certNo: String,
    totalAmount: Number
  },
  data() {
    return {
      visible: true,
      loading: true,
      psnInfo: null,
      preSettleData: null,
      needOtherPay: false,
      payMethod: '',
      baseUrl: 'http://172.20.10.2:10009'
    }
  },
  mounted() {
    this.queryPsnInfo()
  },
  methods: {
    queryPsnInfo() {
      axios.post(`${this.baseUrl}/mi/queryPsnInfo`, {
        certno: this.certNo
      }).then(res => {
        if (res.data.code === 200) {
          this.psnInfo = res.data.data
        } else {
          Message.error(res.data.message || '查询医保信息失败')
        }
      }).catch(() => {
        Message.error('请求失败')
      }).finally(() => {
        this.loading = false
      })
    },
    preSettle() {
      if (!this.psnInfo) {
        this.$message.error('医保信息未加载');
        return;
      }
      console.log({
        insuplc_admdvs: this.psnInfo.insuplc_admdvs,
        mdtrt_cert_no: this.certNo,
        medfee_sumamt: parseFloat(this.totalAmount),
        psn_no: this.psnInfo.psn_no
      })
      axios.post(`${this.baseUrl}/mi/settleAccountsPre`, {
        insuplc_admdvs: this.psnInfo.insuplc_admdvs,
        mdtrt_cert_no: this.certNo,
        medfee_sumamt: parseFloat(this.totalAmount),
        psn_no: this.psnInfo.psn_no
      }).then(res => {
        if (res.data.code === 200) {
          this.preSettleData = res.data.data
          this.needOtherPay = res.data.data.acct_pay < res.data.data.psn_part_amt
        } else {
          Message.error(res.data.message || '医保预结算失败')
        }
      }).catch(() => {
        Message.error('请求失败')
      })
    },
    doSettle() {
      axios.post(`${this.baseUrl}/mi/settleAccounts`, {
        insuplc_admdvs: this.psnInfo.insuplc_admdvs,
        mdtrt_cert_no: this.certNo,
        medfee_sumamt: parseFloat(this.totalAmount),
        psn_no: this.psnInfo.psn_no
      }).then(res => {
        if (res.data.code === 200) {
          Message.success('医保结算成功')
          this.visible = false
          this.$emit('close')
        } else {
          Message.error(res.data.message || '医保结算失败')
        }
      }).catch(() => {
        Message.error('请求失败')
      })
    },
    confirmOtherPay() {
      if (!this.payMethod) {
        Message.warning('请选择支付方式')
        return
      }
      Message.success(`使用${this.payMethod}完成支付`)
      this.visible = false
      this.$emit('close')
    },
    handleClose() {
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
p {
  margin: 8px 0;
}
</style>
