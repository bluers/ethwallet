<template>
  <div>
    <div class="btn-wrapper">
      <Button class="button" @click="openModal('create_wallet')">创建新钱包</Button>
      <Modal v-model="modal.create_wallet" width="360" :closable="false" :mask-closable="false">
        <p slot="header" style="text-align:center">
            <span>创建新钱包</span>
        </p>
        <div style="text-align:center">
            <i-input v-model="user_entropy" placeholder="请输入一串随机的字符串，以随机生成助记词" style="width: 100%"></i-input>
        </div>
        <div slot="footer" style="text-align:center;">
            <i-button class="button" @click="proceedCreateToPassword">创建</i-button>
            <i-button class="button" @click="closeModal()">关闭</i-button>
        </div>
      </Modal>
      <Modal v-model="modal.password_create" width="360" :closable="false" :mask-closable="false">
        <div style="text-align:center">
            <p style="text-align:left">你的新钱包助记词是： "<span class="danger" v-text="seed"></span>"。温馨提示：请写在纸上并妥善保管，您将需要它来访问钱包。不要让任何人看到这段助记词，否则将存在巨大的数字资产安全风险。确认您已经将助记词写在纸上并妥善保管，并在下方输入一个密码，完成新钱包的创建并牢记这个密码，它将用以保护您的私钥。
            </p>
            <i-input type="password" v-model="user_password" placeholder="请输入密码" style="width: 100%"></i-input>
        </div>
        <div slot="footer" style="text-align:center;">
            <i-button class="button" :loading="modal_loading" @click="createWallet">确定</i-button>
            <i-button class="button" @click="closeModal('password_create')">关闭</i-button>
        </div>
      </Modal>
      <Modal v-model="modal.restore_wallet" width="360" :closable="false" :mask-closable="false">
        <p slot="header" style="text-align:center">
            <span>恢复钱包</span>
        </p>
        <div style="text-align:center">
            <i-input v-model="seed" placeholder="请输入助记词" style="width: 100%"></i-input>
        </div>
        <div slot="footer" style="text-align:center;">
            <i-button class="button" @click="proceedStoreToPassword">确定</i-button>
            <i-button class="button" @click="closeModal()">关闭</i-button>
        </div>
      </Modal>
        <Modal v-model="modal.watch_wallet" width="360" :closable="false" :mask-closable="false">
            <p slot="header" style="text-align:center">
                <span>只读钱包</span>
            </p>
            <div style="text-align:center">
                <span>您将只能查看余额，而无法向外转账</span>
                <i-input v-model="readonly_address" placeholder="请输入钱包地址" style="width: 100%"></i-input>
            </div>
            <div slot="footer" style="text-align:center;">
                <i-button class="button" @click="proceedStoreToAddress">确定</i-button>
                <i-button class="button" @click="closeModal()">关闭</i-button>
            </div>
        </Modal>
      <Modal v-model="modal.password_restore" width="360" :closable="false" :mask-closable="false">
        <p slot="header" style="text-align:center">
            <span>恢复钱包</span>
        </p>
        <div style="text-align:center">
            <i-input type="password" v-model="user_password" placeholder="请输入密码" style="width: 100%"></i-input>
        </div>
        <div slot="footer" style="text-align:center;">
            <i-button class="button" :loading="modal_loading" @click="restoreWallet">确定</i-button>
            <i-button class="button" @click="closeModal()">关闭</i-button>
        </div>
      </Modal>
      <Modal v-model="modal.seed_export" width="360" :closable="false" :mask-closable="false">
        <p slot="header" style="text-align:center">
            <span>请牢记您的助记词，写在纸上并妥善保管</span>
        </p>
        <div style="text-align:center">
          <p class="seed-export">{{seed}}</p>    
        </div>
        <div slot="footer" style="text-align:center;">
            <i-button class="button" @click="closeModal()">关闭</i-button>
        </div>
      </Modal>
      <Modal v-model="modal.password_export" width="360" :closable="false" :mask-closable="false">
        <p slot="header" style="text-align:center">
            <span>备份钱包</span>
        </p>
        <div style="text-align:cente">
            <i-input type="password" v-model="user_password" placeholder="请输入密码" style="width: 100%"></i-input>
        </div>
        <div slot="footer" style="text-align:center;">
            <i-button class="button" :loading="modal_loading" @click="exportWallet">确定</i-button>
            <i-button class="button" @click="closeModal()">关闭</i-button>
        </div>
      </Modal>
      <i-button class="button" @click="openModal('restore_wallet')">恢复钱包</i-button>
      <i-button class="button" @click="openModal('watch_wallet')">只读钱包</i-button>
      <!-- <i-button class="button">帮助</i-button> -->
    </div>r
    <div class="filter-wrapper">
    </div>
    <div class="content-wrapper">
      <ul class="wallet-list">
        <li class="wallet-item" v-for="wallet in wallet_list" :key="wallet.address">
          <div class="wallet-wrapper">
            <h1 class="wallet-address">
              <i class="icon iconfont icon-key"></i>
              <span v-text="wallet.address"></span>
                <img src="../assets/copy.png" class="icon icon-address-copy"  @click="processTransaction(wallet)"/>
                <span style="font-size: 12px;">总交易数:{{wallet.nonce[0]}}</span>
              <p>
              <span class="token-wrapper" v-for="(token, index) in wallet.balances" v-bind:key="index">
                <span>{{token.balance}} {{token.symbol}}</span>
              </span>
              </p>
            </h1>
          </div>
          <button class="button" v-if="wallet && wallet.keystore" @click="proceedExport(wallet)">备份钱包(导出助记词)</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import lightwallet from "eth-lightwallet-jh";
import HookedWeb3Provider from "hooked-web3-provider";
import _ from "lodash";
import dbUtils from "../dbUtils";
import reportUtils from "../reportUtils";
import web3Utils from "../web3Utils";

export default {
  data() {
    return {
      modal: {},
      modal_loading: false,
      user_entropy: "",
      user_password: "",
      wallet_list: [],
      seed: "",
      readonly_address:""
    };
  },
  mounted() {
    this.loadWallet();
    setTimeout(this.updateBalances, 15000)
  },
  methods: {
    openModal(modalname) {
      this.modal = {};
      this.modal[modalname] = true;
    },
    closeModal(modalname) {
      let modal_map = JSON.parse(JSON.stringify(this.modal));
      modalname ? (modal_map[modalname] = false) : (modal_map = {});
      this.modal = modal_map;
      this.modal_loading = false;
      this.user_entropy = "";
      this.user_password = "";
      this.seed = "";
    },

    newAddresses(password, keystore) {
      let _this = this,
        address;
      keystore.keyFromPassword(password, function(err, pwDerivedKey) {
        if (err) {
          reportUtils.report(err);
          return;
        }
        keystore.generateNewAddress(pwDerivedKey, 1);
        _.each(keystore.getAddresses(), address => {
          _this.updateWallet(
            _this.getBalance({
              address: address,
              keystore: keystore
            })
          );
        });
      });
    },
    proceedCreateToPassword() {
      if (!this.user_entropy) {
        this.$Message.error("输入字符先");
      } else {
        this.openModal("password_create");
        this.seed = lightwallet.keystore.generateRandomSeed(this.user_entropy);
      }
    },
    createWallet() {
      if (this.modal_loading) return;

      if (!this.user_password) {
        this.$Message.error("输入密码");
      }

      let _this = this,
        password = this.user_password;

      this.modal_loading = true;

      lightwallet.keystore.createVault(
        {
          password: password,
          seedPhrase: _this.seed,
          //random salt
          hdPathString: "m/0'/0'/0'"
        },
        function(err, ks) {
          if (err) {
            reportUtils.report(err);
            this.$Message.error("创建失败");
            return;
          }
          try {
            _this.newAddresses(password, ks);
            web3Utils.setWebProvider(ks);
          } catch (err) {
            reportUtils.report(err);
            _this.$Message.error("创建失败");
          } finally {
            _this.closeModal();
          }
        }
      );
    },
    updateBalances(displayError){
        var _this = this;
        var web3 = web3Utils.getWeb3()
        this.$root.currentView == 'wallet'  && _this.wallet_list && _this.wallet_list.map(function (_wallet) {

            var wallet = _wallet;

            web3.eth.getTransactionCount(wallet.address, function (err, result){
                wallet.nonce[0] = result;
            });

            wallet.balances && _wallet.balances.map(function (_token, i) {
                var token = _token;

                if(token == null || typeof(token) == 'undefined')return;

                if(token.address == "ETH"){
                    web3.eth.getBalance(_wallet.address, function (err, result) {
                        if (err) {
                            reportUtils.report(e);
                            if(displayError)_this.$Message.error("获取余额失败");
                            return
                        }
                        //console.log(_wallet.address);
                        token.balance = web3Utils.toRealAmount(result);
                        wallet.balances[i] = token;
                    })
                }
                else{

                    let
                        erc20tokens = web3Utils.getErc20Tokens(),
                        erc20token = _.find(erc20tokens, {
                            address: token.address
                        });
                    var contract = erc20token.contract;
                    token.decimals = erc20token.decimals;

                    contract && contract.balanceOf("" + _wallet.address, function (err, balance) {
                        if(err){
                            reportUtils.report(e);
                            if(displayError)_this.$Message.error("获取余额失败");
                            return
                        }
                        //console.log(_wallet.address+", "+balance.toString());
                        token.balance = web3Utils.toRealAmount(
                            balance,
                            token.decimals
                        )
                        wallet.balances[i] = token;
                    });
                }
            })
        });

        if(displayError){
            return
        }
        else{
            setTimeout(function() { _this.updateBalances() }, 10000);
        }
    }
    ,
    getBalance(wallet) {
      web3Utils.setWebProvider(wallet.keystore);

      var _this = this,
        web3 = web3Utils.getWeb3(),
        erc20tokens = web3Utils.getErc20Tokens(),
        _wallet = _.defaults({}, wallet),
        _token = {
          address: "ETH",
          symbol: "ETH"
        };

      _wallet.balances = [];
      _wallet.nonce = []

      try {

          web3.eth.getTransactionCount(_wallet.address, function (err, result){
              _wallet.nonce.push(result);
          });

          web3.eth.getBalance(_wallet.address, function (err, result) {
              if(err){
                  reportUtils.report(err);
                  _this.$Message.error("获取余额失败");
                  return
              }
              _token.balance = web3Utils.toRealAmount(result);
              _wallet.balances.push(_token);

              _.forEach(erc20tokens, (token, index) => {
                  var _token = {
                      address: token.address,
                      symbol: token.symbol,
                      decimals: token.decimals,
                      balance: "...",
                      //contract: token.contract
                  };
                  _wallet.balances.push(_token);

                  token.contract.balanceOf("" + _wallet.address, function (err, balance) {
                      if(err){
                          reportUtils.report(err);
                          _this.$Message.error("获取余额失败");
                          return
                      }

                      var _token = _wallet.balances[index+1];
                      _token.balance =web3Utils.toRealAmount(
                              balance,
                              token.decimals
                          );

                      _wallet.balances[index+1] = _token;
                  });
              });
          });

          this.updateWallet(wallet);
      }
      catch (e) {
          reportUtils.report(e);
          _this.$Message.error("获取余额失败");
      }
      return _.defaults({}, wallet, _wallet);
    },
    proceedStoreToPassword() {
      if (!this.seed) {
        this.$Message.error("输入seed");
      } else {
        this.openModal("password_restore");
      }
    },
      proceedStoreToAddress() {
        var web3 = web3Utils.getWeb3();
        var address = this.readonly_address;
        try {
            if(address.startsWith("0x") || address.startsWith("0X")){
                address = address.slice(2);
            }
            var iban = web3.eth.iban.fromAddress(address);
            var _this = this;

            if (iban && web3.eth.iban.isValid(iban.toString())) {
                _this.updateWallet(
                    _this.getBalance({
                        address: "0x"+address,
                        keystore: null
                    })
                );
                _this.loadWallet();
                _this.closeModal();
            }
            else {
                _this.$Message.error("错误的地址");
            }
        }catch(err){
            _this.$Message.error("错误的地址");
        }
      },
    restoreWallet() {
      var _this = this,
        password = this.user_password,
        seed = this.seed;

      this.modal_loading = true;

      if (lightwallet.keystore.isSeedValid(seed)) {
        lightwallet.keystore.createVault(
          {
            password: password,
            seedPhrase: seed,
            //random salt
            hdPathString: "m/44'/60'/0'/0"
          },
          function(err, ks) {
            if (err) {
              window.location.reload();
            }
            try {
              _this.newAddresses(password, ks);
              web3Utils.setWebProvider(ks);
              setTimeout(_this.loadWallet, 1000);
            } catch (e) {
              reportUtils.report(e);
              _this.$Message.error("恢复失败");
            } finally {
              _this.closeModal();
            }
          }
        );
      } else {
        _this.$Message.error("无效的Seed");
        _this.closeModal();
      }
    },
    updateWallet(wallet) {
      let index = _.findIndex(this.wallet_list, ["address", wallet.address]);
      if (index != -1) {
        // lightwallet.keystore.upgradeOldSerialized(wallet.keystore, password, function(keystore){
        //   wallet.keystore = keystore;
        // })
        this.wallet_list[index] = wallet;
      } else {
        this.wallet_list.push(wallet);
        var addresses = dbUtils.get("address_list");

        if(addresses.indexOf(wallet.address) < 0){
            dbUtils.set(
                "address_list",
                [addresses, wallet.address].join(" ")
            );
        }
      }
      if(wallet.keystore)dbUtils.set(wallet.address, wallet.keystore.serialize());
      this.$root.globalData.wallet_list = this.wallet_list;
    },
    loadWallet() {
      let _this = this;
      this.wallet_list = this.$root.globalData.wallet_list.map(wallet => {
        return _this.getBalance(wallet);
      });
    },
    proceedExport(wallet) {
      this.openModal("password_export");
      this.current_wallet = wallet;//_.cloneDeep(wallet);
    },
    exportWallet() {
      let _this = this,
        password = this.user_password,
        keystore = undefined
        //_.find(this.wallet_list, this.current_wallet).keystore; //_.find causes problems in offline mode
        for(var i=0;i<this.wallet_list.length;i++){
          if(this.wallet_list[i].address == this.current_wallet.address){
              keystore = this.wallet_list[i].keystore
          }
        }

      keystore.keyFromPassword(password, function(err, pwDerivedKey) {
        if (err) {
          reportUtils.report(err);
          _this.$Message.error(err);
          return;
        }
        try {
          _this.seed = keystore.getSeed(pwDerivedKey);
          _this.openModal("seed_export");
        } catch (e) {
          _this.$Message.error("导出失败");
        }
      });
    },
    showtooltip(tip, e){

      var tooltip = document.createElement('div')
      tooltip.style.cssText =
          'position:absolute; background:black; color:white; padding:4px;z-index:10000;'
          + 'border-radius:2px; font-size:12px;box-shadow:3px 3px 3px rgba(0,0,0,.4);'
          + 'opacity:0;transition:opacity 0.3s'
      tooltip.innerHTML = tip || '已复制!'
      document.body.appendChild(tooltip)

        var evt = e || event

        tooltip.style.left = evt.pageX - 10 + 'px'
        tooltip.style.top = evt.pageY + 15 + 'px'
        tooltip.style.opacity = 1
        setTimeout(function(){
            tooltip.style.opacity = 0
            document.body.removeChild(tooltip)
        }, 500)
    },
    processTransaction(wallet) {
      //this.$root.globalData.current_wallet = _.cloneDeep(wallet);

        function selectElementText(el){
            var range = document.createRange() // create new range object
            range.selectNodeContents(el) // set range to encompass desired element text
            var selection = window.getSelection() // get Selection object from currently user selected text
            selection.removeAllRanges() // unselect any user selected text (if any)
            selection.addRange(range) // add range to Selection object to select it
        }

        selectElementText(event.target.parentElement.children[1]);
        document.execCommand("copy");

        this.showtooltip("复制成功!")
      //window.location.hash = "send";
    }
  }
};
</script>

<style lang="less" scoped>
.seed-export {
  font-size: 30px;
  color: red;
}

.token-wrapper {
  margin-right: 10px;
  font-size: 14px;
  color: #ffffff;
}
.wallet-list {
  display: flex;
  flex-direction: column;
  .wallet-item {
    display: inline-flex;
    flex-direction: row;
    flex-wrap: nowrap;
    align-items: center;
    height: 70px;
    padding: 0 20px;
    background: rgba(46, 46, 46, 1);
    margin-bottom: 10px;
    border: 1px solid rgb(204, 204, 204);
    .wallet-wrapper {
      flex-grow: 1;
      .wallet-address {
        color: #ccc;
        font-size: 20px;
      }
    }
      .icon-address-copy{
          width:20px;
          height:20px;
          margin-top:5px;
      }
    .export {
      text-align: right;
      width: 120px;
      flex-direction: row;
      font-size: 12px;
      color: #fff;
      //visibility: hidden;
      cursor: pointer;
    }
    &:hover,
    &.active {
      background: rgb(46, 46, 46);
      border: 1px solid #fff;
      .wallet-name {
        color: #efefef;
      }
      .wallet-address {
        color: #fff;
      }
    }
    &:hover {
      .export {
        visibility: visible;
        color: #fff;
      }
    }
  }
}
</style>
