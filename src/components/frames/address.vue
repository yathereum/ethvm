<template>
  <div class="">
    <div class="container">
      <!-- Page Title -->
      <div class="page-title-container">
        <div class="page-title">
          <h3>Address </h3>
          <h6 class="text-muted">Ethereum wallet address overview / Transactions information</h6>
        </div>
        <div class="search-block">
          <block-search></block-search>
        </div>
        <!-- End Page Title -->
      </div>
      <!-- Address Details -->
      <div class="row">
        <div class="col-md-12 col-sm-12 col-xs-12">
          <address-detail :account="account"></address-detail>
        </div>
        <!-- End Address Details -->
      </div>
      <!-- Tab Menu -->
      <tab-component :tabs="addressTabs"></tab-component>
      <!-- End Tab Menu -->
      <!--Tab Content -->
      <div class="tab-menu-container">
        <!-- Transactions -->
        <div v-if="addressTabs[0].isActive">
          <div v-if="account.txs">
            <block-address-tx :address='account' :transactions='account.txs'></block-address-tx>
          </div>
          <!-- End Transactions -->
        </div>
        <!-- Tokens -->
        <div v-if="addressTabs[1].isActive">
          <div v-if="!tokenError">
            <div v-if="tokensLoaded">
              <block-token-tracker :tokens="account.tokens" :holder="account.address"></block-token-tracker>
            </div>
            <div v-else class="loading-tokens">
              <i class="fa fa-spinner fa-pulse fa-4x fa-fw"></i>
              <span class="sr-only">Loading...</span>
            </div>
          </div>
          <div v-else class="info">
            <p> Oops Something Went Wrong :( </p>
          </div>
          <!-- End Tokens -->
        </div>
        <!-- Pending Transactions -->
        <div v-if="addressTabs[2].isActive" class="">
          <block-address-tx :address='account' :transactions='account.txs' :isPending=true></block-address-tx>
          <!-- End Pending Transactions -->
        </div>
        <!--Mining History -->
        <div v-if="account.isMiner">
          <div v-if="addressTabs[3].isActive">
            <div>
              <ul>
                <li>Name:</li>
                <li>TWN</li>
                <li>Balance:</li>
                <li>20,930 TWN</li>
                <li>Value:</li>
                <li>$0.00</li>
                <li>ERC 20 Contract:</li>
                <li>0x045619099665fc6f661b1745e5350290ceb933f</li>
              </ul>
            </div>
          </div>
          <!--End Mining History -->
        </div>
        <!-- End Tab Content -->
      </div>
    </div>
    <!-- container -->
  </div>
</template>
<script lang="ts">
import Vue from "vue";
import bn from "bignumber.js";
import {
  common,
  Tx
} from "@/libs";
import ethUnits from "ethereumjs-units";
var utils = require("../../libs/utils.js");
const MAX_ITEMS = 20;
export default Vue.extend({
  name: "FrameAccount",
  props: ["address", "totalTxs", "tokens", "txs"],
  data() {
    return {
      account: {
        address: this.address,
        balance: 0,
        balanceUSD: 0,
        ethusd: 0,
        totalTxs: this.totalTxs,
        tokens: this.tokens,
        txs: this.txs,
        isMiner: false
      },
      addressTabs: [{
        id: 0,
        title: 'Transactions',
        isActive: true,
      }, {
        id: 1,
        title: 'Tokens',
        isActive: false,
      }, {
        id: 2,
        title: 'Pending Transactions',
        isActive: false,
      }],
      tokensLoaded: false,
      tokenError: false,
      usdValue: {
        ETH: {
          value: 0
        }
      }
    };
  },
  created() {
    var _this = this;
    /* Geting Address Balance: */
    this.$socket.emit("getBalance", this.address, (err, result) => {
      if (!err && result) {
        let balance = common
          .EthValue(common.HexToBuffer(result.result))
          .toEth();
        _this.account.balance = balance;
      }
    });
    /* Getting Token Balances: */
    this.$socket.emit("getTokenBalance", this.address, (err, result) => {
      if (result != null) {
        _this.account.tokens = result
        _this.tokensLoaded = true;
        console.log("tokens", _this.account.tokens)
      } else {
        _this.tokenError = true;
      }
    });
    /* Getting Total Number of Tx: */
    this.$socket.emit("getTotalTxs", this.address, (err, result) => {
      _this.account.totalTxs = result;
    });
    /*Getting USD Values: */
    this.$socket.emit("getTokenToUSD", [], (err, result) => {
      _this.account.ethusd = result[0][1];
      _this.usdValue.ETH.value = result[0][1];
    });
    /*Getting Address Transactions: */
    this.$socket.emit("getTxs", this.address, (err, result) => {
      var txs = [];
      result.forEach(element => {
        txs.push(new Tx(element));
      });
      _this.account.txs = txs;
    });
    _this.setTabs();
    /*Getting Address Pending Transactions: */
    // Method here:
  },
  methods: {
    /*Checking of address is Miner? --> add new tab for the manu*/
    setTabs() {
      let _this = this
      if (_this.account.isMiner) {
        let newTab = {
          id: 3,
          title: 'Mining History',
          isActive: false,
        }
        this.addressTabs.push(newTab)
      }
    }
  },
  computed: {}
});
</script>
<style scoped="" lang="less">
@import "~lessPath/sunil/frames/address.less";
</style>
