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

      <div v-if="preSettleData" style="margin-top: 20px;">

        <h4>医保预结算信息</h4>
        <el-row :gutter="20">
          <el-col :span="12">
            <p><b>结算时间：</b>{{ preSettleData.setl_time }}</p>
            <p><b>出生日期：</b>{{ preSettleData.brdy }}</p>
            <p><b>民族：</b>{{ preSettleData.naty }}</p>
            <p><b>证件类型：</b>{{ preSettleData.mdtrt_cert_type }}</p>
            <p><b>证件号：</b>{{ preSettleData.certno }}</p>
            <p><b>性别：</b>{{ preSettleData.gend === '1' ? '女' : '男' }}</p>
            <p><b>年龄：</b>{{ preSettleData.age }}</p>
            <p><b>医保类型：</b>{{ preSettleData.insutype }}</p>
            <p><b>个人账户支付：</b>¥{{ preSettleData.acct_pay }}</p>
            <p><b>个人现金支付：</b>¥{{ preSettleData.psn_cash_pay }}</p>
            <p><b>个人账户余额：</b>¥{{ preSettleData.balc }}</p>
          </el-col>

          <el-col :span="12">
            <!-- <p><b>居民大病保险资金支出：</b>¥{{ preSettleData.hifmi_pay }}</p>
            <p><b>公务员医疗补助资金支出：</b>¥{{ preSettleData.cvlserv_pay }}</p>
            <p><b>伤残人员医疗保障基金支出：</b>¥{{ preSettleData.hifdm_pay }}</p>
            <p><b>职工大额医疗费用补助基金支出：</b>¥{{ preSettleData.hifob_pay }}</p>
            <p><b>企业补充医疗保险基金支出：</b>¥{{ preSettleData.hifes_pay }}</p> -->
            <p><b>医疗救助基金支出：</b>¥{{ preSettleData.maf_pay }}</p>
            <p><b>其他支出：</b>¥{{ preSettleData.oth_pay }}</p>
            <p><b>基本医疗保险统筹基金支出：</b>¥{{ preSettleData.hifp_pay }}</p>
            <p><b>基金支付总额：</b>¥{{ preSettleData.fund_pay_sumamt }}</p>
            <p><b>全自费金额：</b>¥{{ preSettleData.fulamt_ownpay_amt }}</p>
            <p><b>超限价自费费用：</b>¥{{ preSettleData.overlmt_selfpay }}</p>
            <p><b>先行自付金额：</b>¥{{ preSettleData.preselfpay_amt }}</p>
            <p><b>符合政策范围金额：</b>¥{{ preSettleData.inscp_scp_amt }}</p>
            <p><b>实际支付起付线：</b>¥{{ preSettleData.act_pay_dedc }}</p>
            <p><b>基本医疗保险统筹基金支付比例：</b>¥{{ preSettleData.pool_prop_selfpay }}</p>
            <p><b>个人负担总金额：</b>¥{{ preSettleData.psn_part_amt }}</p>
          </el-col>
        </el-row>

        <el-button type="success" @click="doSettle" style="margin-top: 20px;">医保结算</el-button>

        <div v-if="needOtherPay" style="margin-top: 20px;">
          <p>医保余额不足，还需支付：<b style="color: red">¥{{ remainingAmount }}</b></p>
          <el-radio-group v-model="payMethod">
            <el-radio label="微信">微信支付</el-radio>
            <el-radio label="支付宝">支付宝支付</el-radio>
            <el-radio label="现金">现金支付</el-radio>
          </el-radio-group>
          <el-button type="primary" @click="confirmOtherPay" style="margin-top: 10px;">确认支付</el-button>
        </div>
        <div v-if="alipayImg" style="text-align:center;margin-top:20px">
          <p>请使用支付宝扫描二维码支付 ¥{{ remainingAmount }}</p>
          <img :src="alipayImg" alt="支付宝二维码" style="width:200px;height:200px" />
          <p>支付完成后系统会自动确认</p>
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
      payMethod: '',
      preSettleResult: null,
      needOtherPay: false,
      baseUrl: 'http://172.20.10.2:10009',
      remainingAmount: 0,
      alipayImg: '',           // 支付二维码图片
      outTradeNo: ''
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
          const result = res.data.data
          const total = result.medfee_sumamt         // 总金额
          const fundPay = result.fund_pay_sumamt     // 医保支付
          const personPay = result.psn_part_amt      // 个人自付
          const cashPay = result.psn_cash_pay        // 需补的现金

          console.log('医保结算返回:', result)

          if (cashPay <= 0) {
            // 医保已完全覆盖，不需要再支付
            this.$message.success('医保结算成功（全额）')
            this.visible = false
            this.$emit('settle-success') // 通知父组件进行缴费登记
          } else {
            // 医保结算成功，但还需自付部分
            this.remainingAmount = cashPay
            this.needExtraPay = true     // 显示“请选择其他支付方式”区域
            this.$message.info(`医保结算成功，仍需自费 ¥${cashPay}，请选择支付方式`)
          }
        } else {
          this.$message.error(res.data.message || '医保结算失败')
        }
      }).catch(() => {
        this.$message.error('医保结算接口请求失败')
      })
    },
    async confirmOtherPay() {
      if (!this.payMethod) {
        this.$message.warning('请选择支付方式')
        return
      }

      // 生成 outTradeNo：日期 + 随机数 + 结算金额，确保唯一
      const today = new Date()
      const dateStr = today.toISOString().slice(0, 10).replace(/-/g, '')
      const rand = Math.floor(1000 + Math.random() * 9000)
      this.outTradeNo = `${dateStr}${rand}`

      try {
        // 调用支付宝二维码接口
        const res = await axios.post(`${this.baseUrl}/pay/payQuery`, {
          money: this.remainingAmount,
          name: '医保',
          outTradeNo: this.outTradeNo,
          type: 'alipay'
        })
        console.log("tradeno,", this.outTradeNo)

        if (res.data.code === 200 && res.data.data.img) {
          this.alipayImg = res.data.data.img
          this.$message.success(`已选择 ${this.payMethod}，请扫码完成支付 ¥${this.remainingAmount}`)
          this.startPollingPayStatus()
        } else {
          this.$message.error('生成支付宝二维码失败，请稍后重试')
        }
      } catch (err) {
        console.error(err)
        this.$message.error('调用支付接口失败')
      }
    },
    startPollingPayStatus() {
      let attempt = 0
      const maxAttempts = 10
      const interval = setInterval(async () => {
        attempt++

        try {
          const res = await axios.post(`${this.baseUrl}/pay/payRecord`, {
            money: this.remainingAmount,
            name: '医保',
            outTradeNo: this.outTradeNo,
            type: 'alipay'
          })

          if (res.data.code === 200 && res.data.data.status === 1) {
            clearInterval(interval)
            this.$message.success('自费支付成功')
            this.visible = false
            this.$emit('settle-success')
          } else if (attempt >= maxAttempts) {
            clearInterval(interval)
            this.$message.warning('支付超时，请刷新页面重新支付')
          }
        } catch (err) {
          console.error('轮询支付失败', err)
        }
      }, 3000) // 每 3 秒轮询一次
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
