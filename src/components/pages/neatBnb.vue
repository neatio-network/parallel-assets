<template>
  <div class="main">

    <div v-if="step == 1">
      <Access @unlock="unlock"></Access>
    </div>
    <div v-if="step == 2" style="padding-bottom: 90px">


      <div class="box0" v-show="address != null && currentChainId != '0x203'">
        <div class="ntrk" v-show="address != null && currentChainId != '0x203'">
          <div>Please switch to Neatio Chain</div>
        </div>
        <div
          class="neat-logo"
          v-show="address != null && currentChainId == '0x203'"
        >
          <img src="../../assets/neat.png" alt="Neat logo" class="neatLogo" />
        </div>

        <div
          class="wallet-address"
          v-show="address != null && currentChainId == '0x203'"
        >
          <div
            class="address-title"
            v-show="address != null && currentChainId == '0x203'"
          >
            <span class="noDisplay" style="color: #7192b3; font-weight: bold"
              >Address: {{ address }}
            </span>
          </div>

          <span class="displayIt" style="color: white">{{ shortAddress }}</span>

          <div
            class="address-title"
            v-show="address != null && currentChainId == '0x203'"
          >
            <span style="color: #7192b3; font-weight: bold"></span
            ><span style="color: white">{{ balance }}</span>
            <span style="color: #00ffff"> BNB</span>
          </div>


        </div>
      </div>

      <div class="box2" v-show="address != null && currentChainId == '0x203'">

        <div
          class="neat-logo"
          v-show="address != null && currentChainId == '0x203'"
        >
          <img src="../../assets/neat.png" alt="Neat logo" class="neatLogo" /> 
          <span  style="color: #00ffff;  font-size: 24px; margin-bottom: 5px;"> Neatio Network</span>
        </div>

        <div
          class="wallet-address"
          v-show="address != null && currentChainId == '0x203'"
        >
          <div
            class="address-title"
            v-show="address != null && currentChainId == '0x203'"
          >
            <span class="noDisplay" style="color: #7192b3; font-weight: bold"
              >Address: {{ address }}
            </span>
          </div>

          <span class="displayIt" style="color: white">Address: {{ shortAddress }}</span>

          <div
            class="address-title"
            v-show="address != null && currentChainId == '0x203'"
          >
            <span style="color: #7192b3; font-weight: bold"></span
            ><span style="color: #00ffff">Balance:</span>
            <span style="color: #00bfff">{{ balance }}</span>
            <span style="color: #00ffff"> NEAT</span>
          </div>

        </div>
        <div class="info-box"></div>

        <div class="itemNeat">
          <p style="font-size: 14px"></p>
          <input
            class="inputs"
            v-model="amountToSwap"
            placeholder="Amount (min. 5000)"
          />
        </div>
        <div class="neatrate-bnb">Swap fee 100 NEAT</div>
        <div class="neatrate-bnb">( BSC Network fee )</div>

        <div class="btn" v-show="address != null && currentChainId == '0x203'">
          <button id="gtButton" @click="neatSwap">{{ "SWAP" }}</button>
        </div>
      </div>

      <div class="noteText">
       <div class="dashboard1">
        <div class="dashboard4">
          <span style="color: white">NOTE:</span> The wrapped NEAT tokens will be available on the BSC Network after 200 confirmations on both chains. 
          This can take up to 8h.
        </div>
        <div class="dashboard4">
          <span style="color: white">T&C:</span> By visiting and using the
          Neatio websites, you hereby agree with our terms and conditions. Check them
          the bottom of this page.
        </div>

       </div>


      </div>
    </div>
  </div>
</template>

<script>
import Access from "./modules/access";
import EyeInput from "./modules/eyeInput";
import neatioapi from "neatioapi";
import axios from "axios";
const Utils = neatioapi.utils;
const Web3 = require("web3");
const web3 = new Web3("https://rpc.neatio.net");
export default {
  data() {
    return {
      curNav: "Home",
      searchContent: "",
      otherSearch: "",
      currentChainId: '',
      chainId: '0x203',
      step: 2,
      balance: "",
      fullbalance: "",
      address: null,
      addresss: null,
      shortAddress: null,
      privateKey: "",
      currentChainId: "",
      chainID: "0x203",
      staking: "",
      rewards: "",
      amount: "",
      limit: "",
      price: "",
      amountToSwap: "",
      amountBNB: "",
      bnbprice: "",
      totalNEAT: "",
      totalBNB: "",
      neatPrice: "",
      neatToSub: "",
    };
  },
  components: {
    Access,
    EyeInput,
  },


  mounted() {
    this.connectAccount();
    this.initialize();
    this.checkWallet();

  },
  methods: {
    initialize() {
      ethereum.on("chainChanged", (_chainId) => {
        this.getGasPrice();
        this.getBalance();
      });
      ethereum.on("accountsChanged", (_accounts) => {
        this.address = _accounts[0];
        this.getBalance();
      });
    },
    checkWallet() {
      if (this.walletNF == null) {
        console.log("not found");
      } else {
        console.log("wallet found");
      }
    },
    strTrim(str) {
      str = str + "";
      return str.replace(/(^\s*)|(\s*$)/g, "");
    },
    highlight(index) {
      this.curNav = index;
    },
    getLocaction() {
      this.isTestNetwork =
        window.location.hostname.substr(0, 4) === "test" ||
        window.location.hostname.substr(0, 4) === "loca";
    },
    async initialize() {
      this.currentChainId = await ethereum.request({ method: "eth_chainId" });
      ethereum.on("chainChanged", (_chainId) => {
        this.connectAccount(_chainId);
      });
      ethereum.on("accountsChanged", (_accounts) => {
        this.requestAccount();
      });
      this.requestAccount();
    },
    refresher() {
    },
    async requestAccount() {
      this.currentChainId = await ethereum.request({ method: "eth_chainId" });
      try {
        if (this.currentChainId !== this.chainId) {
          this.connectAccount();
        } else {
          this.addresss = `BSC Network`;
          this.shortAddress = `${accounts[0].substr(
            0,
            6
          )}...${accounts[0].slice(-4)}`;
        }
      } catch (e) {
        console.log("request accounts error:");
      }
    },
    async connectAccount() {
      try {
        const accounts = await ethereum.request({
          method: "eth_requestAccounts",
        });
        this.address = accounts[0];
        this.getBalance();
        this.getGasPrice();
        this.shortAddress = `${accounts[0].substr(0, 6)}...${accounts[0].slice(
          -4
        )}`;
      } catch (e) {
        console.log("request accounts error:", e);
      }
    },


    async switchToBSCChain() {
      let chainIds = "0x203";
      let rpc = "https://rpc.neatio.net";
      let browser = "https://scan.neatio.net";
      let chainName = "Neatio Network";
      try {
        this.currentChainId = await ethereum.request({ method: "eth_chainId" });
        if (this.currentChainId === chainIds) {
          window.alert("Neatio Network has been added to Metamask.");
        }
        await ethereum.request({
          method: "wallet_switchEthereumChain",
          params: [{ chainId: chainIds }],
        });
      } catch (e) {
        if (e.code === 4902) {
          try {
            await ethereum.request({
              method: "wallet_addEthereumChain",
              params: [
                {
                  chainId: chainIds,
                  chainName: chainName,
                  nativeCurrency: {
                    name: "NEAT",
                    symbol: "NEAT",
                    decimals: 18,
                  },
                  rpcUrls: [rpc],
                  blockExplorerUrls: [browser],
                },
              ],
            });
            this.currentChainId = await ethereum.request({
              method: "eth_chainId",
            });
          } catch (e) {
            console.log("add network error", e);
          }
        }
      }
    },
    getBalance() {
      ethereum
        .request({
          method: "eth_getBalance",
          params: [this.address],
        })
        .then((result) => {
          this.balance = Utils.toNEAT(result);
        })
        .catch((error) => {
          console.log("Error", error);
        });
    },
    getGasPrice() {
      ethereum
        .request({
          method: "eth_gasPrice",
          params: [],
        })
        .then((result) => {
          console.log("gasprice", result);
          this.price = Utils.toNEAT(result);
        })
        .catch((error) => {
          console.log("error", error);
        });
    },

    async getNeatPrice() {
      const cpURL = "https://api.coinpaprika.com/v1/tickers/neat-neatio";
      await axios
        .get(cpURL)
        .then((response) => (this.neatPrice = response.data.quotes.USD.price));
      
      const neatFee = (0.08 / this.neatPrice);

    },

    async neatSwap() {
      if (this.amountToSwap < 5000) {
        this.info("error", this.$t("Swap is min. 5000 NEAT"));
        return;
      }
      let neatToSwap = this.amountToSwap;
      const params = [
        {
          from: this.address,
          to: "0xdd112cb6a0afb5b589c75c144f857d3093f0aff5",
          value: Utils.toHex(Utils.fromNEAT(`${neatToSwap}`)),
          gas: Utils.toHex("21000"),
          gasPrice: Utils.toHex(Utils.fromNEAT(this.price)),
        },
      ];
      ethereum
        .request({
          method: "eth_sendTransaction",
          params,
        })
        .then((result) => {
          console.log("hash", result);
          this.$alert(
            "NOTE:  It can take up to 8h to recieve your Wrapped NEAT.",
            `You succesfully bridged ${this.amountToSwap} NEAT. \n This is neat!`,
            {
              confirmButtonText: this.$t("CLOSE"),
              type: "success",
            }
          );
        })
        .catch((error) => {
          console.log("tx error", error);
        });
    },
  },
};
</script>

<style scoped>
@media only screen and (max-width: 800px) {
  .main {
    width: 80%;
    padding: 10px;
  }
  .right {
    width: 100%;
  }
}
@media only screen and (max-width: 500px) {
  .menu,
  .main,
  .right {
    width: auto;
    padding: 10px;
  }
}
button {
  border: none;
  min-width: 40px;
  font-family: Arial, Helvetica, sans-serif;
  text-transform: uppercase;
  cursor: pointer;
  color: #00ffff;
  box-shadow: inset 0 0 0.1em #00ffff, 0 0 0.1em #00ffff;
  border: #00ffff solid 1px;
  background-color: #24292f;
  border-radius: 4px;
  outline: none;
  margin: 0px 0px 0px 60px;
}
.info {
  display: inline-block;
  margin-left: 10px;
  margin-top: 50px;
}
.ntrk {
  margin: 0 auto;
  justify-content: center;
}
.main {
  padding: 10px;
}
.address-title {
  margin-bottom: 5px;
  margin-top: 10px;
  width: auto;
}
.wallet-address {
  margin-bottom: 5px;
  margin-left: 48px;
  margin-top: 10px;
  width: auto;
  color: #7192b3;
}
.inputs {
  background-color: #000;
  border-radius: 10px;
  border: 1px solid #dcdfe6;
  box-sizing: border-box;
  color: #00ffff;
  display: inline-block;
  font-size: inherit;
  height: 40px;
  width: 100%;
  text-align: center;
}
::-webkit-input-placeholder {
  text-align: center;
}
:-moz-placeholder {
  text-align: center;
}
.dashboard1 {
  margin-top: 120px;
}
.dashboard4 {
  font-size: 14px;
  margin-top: 20px;
  color: #fff;
}
.noteText {
  padding: 40px;
}
.noteText2 {
  margin: 10px;
}
@media only screen and (max-width: 540px) {
  .wallet-address {
    margin-bottom: 5px;
    margin-left: 18px;
    margin-top: 10px;
    width: auto;
    color: #7192b3;
  }
  .dashboard1 {
  margin-top: 0px;
}
  .neat-logo {
    margin-left: 12px;
    margin-top: 32px;
    margin-bottom: 20px;
  }
  .noDisplay {
    display: none;
  }
}
@media only screen and (min-width: 1200px) {
  .displayIt {
    display: none;
  }
}
.neat-logo {
  margin-left: 36px;
  margin-top: 20px;
}
.info-box {
  padding-top: 40px;
  text-align: center;
  margin: 0 auto;
}
.card-text {
  background-color: #000;
  padding-top: 10px;
  padding-bottom: 5px;
}
.neatLogo {
  width: 32px;
  height: 32px;
  margin-left: 10px;
}
.buyimg {
  width: 100vw;
  max-width: 720px;
}
.card-number {
  color: #00ffff;
  margin-top: 10px;
  margin-bottom: 10px;
}
.gt {
  width: 280px;
  height: 44px;
  margin-left: 450px;
  margin-top: 50px;
  color: #000;
}
.button {
  color: #000;
  text-align: center;
  width: 50%;
  margin: 0 auto;
}
.itemNeat {
  width: 50%;
  margin: 0 auto;
  text-align: center;
}
.neatrate {
  text-align: center;
  color: #00ffff;
  margin-bottom: 15px;
  font-size: 24px;
}
.neatrate-bnb {
  text-align: center;
  color: #7192b3;
  margin-top: 5px;
}
.btn {
  display: flex;
}
#gtButton {
  width: 140px;
  height: 40px;
  color: #000000;
  border: 1px solid #000;
  border-radius: 10px;
  background-color: #00ffff;
  font-size: 16px;
  cursor: pointer;
  margin: 0 auto;
  margin-top: 16px;
  font-weight: bold;
}
#gtButton:hover {
  background-color: #00ffff;
}
</style>

<style>
.el-message-box__message p {
  margin: 0 !important;
  line-height: 24px !important;
}
.el-message-box__title {
  text-align: center !important;
  font-family: Arial, Helvetica, sans-serif !important;
}
.el-message-box__message {
  text-align: center !important;
  font-family: Arial, Helvetica, sans-serif !important;
}
</style>