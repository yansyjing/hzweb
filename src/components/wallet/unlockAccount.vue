<template>
    <div>
        <el-form
            label-width="140px"
            label-position="left"
            @submit.native.prevent>

            <el-form-item class="el-wallet-style">
                <div class="wallet-decrypt-type no-select-text" style="position: relative;left:-70px;">
                    <!--<span class="wallet-decrypt-item" :class="{active:form.decryptType==='4'}">-->
                    <!--@click="selectDecryptType('4')" v-show="hasWalletCom">
                  <i class="wallet-decrypt-type-icon icon3" :class="{'icon3-active':form.decryptType==='4'}"></i>
                  钱包文件-->
                    <!--</span>-->
                    <span class="wallet-decrypt-item" :class="{active:form.decryptType==='1'}"
                          @click="selectDecryptType('1')" v-show="!hasWalletCom">
                        <i class="wallet-decrypt-type-icon icon1" :class="{'icon1-active':form.decryptType==='1'}"></i>
                        {{$t('privatekey')}}
                    </span>
                    <span class="wallet-decrypt-item" :class="{active:form.decryptType==='2'}"
                          @click="selectDecryptType('2')" v-show="!hasWalletCom">
                        <i class="wallet-decrypt-type-icon icon2" :class="{'icon2-active':form.decryptType==='2'}"></i>
                        {{$t('accountfile')}}
                    </span>
                    <span class="wallet-decrypt-item" :class="{active:form.decryptType==='3'}"
                          @click="selectDecryptType('3')" v-show="!hasWalletCom">
                        <i class="wallet-decrypt-type-icon icon3" :class="{'icon3-active':form.decryptType==='3'}"></i>
                        {{$t('mnemonic')}}
                    </span>
                </div>
                <!--<p>{{decryptInfo}}</p>-->
            </el-form-item>

            <div v-show="!hasWalletCom && form.decryptType==='1'" style="position: relative;left:-70px;">
                <el-form-item class="el-wallet-style mt-40">
                    <el-input class="el-wallet-account-input"
                              v-model="form.privateKey"
                              :placeholder="$t('enterPrivatekey')"
                              auto-complete="off"
                    ></el-input>
                </el-form-item>
            </div>

            <div v-show="!hasWalletCom && form.decryptType==='2'" style="position: relative;left:-70px;">
                <el-form-item class="el-wallet-style mt-40">
                    <input id="fileUpload1" type="file" style="display:none" @change="uploadFile1">
                    <el-button class="choose-file-btn" @click="selectFile1">{{$t('selectaccountfile')}}</el-button>
                </el-form-item>

                <el-form-item class="el-wallet-style mt-40">
                    <el-input class="el-wallet-input"
                              v-model="form.pwd"
                              type="password"
                              :placeholder="$t('pleaseenteryouraccountpassword')"
                              auto-complete="off"
                    ></el-input>
                </el-form-item>
            </div>

            <div v-show="!hasWalletCom && form.decryptType==='3'" style="position: relative;left:-70px;">
                <el-form-item class="el-wallet-style">
                    <el-input class="el-wallet-account-input"
                              v-model="form.mnemonic"
                              type="textarea"
                              :autosize="{ minRows: 3, maxRows: 3 }"
                              resize="none"
                              :placeholder="$t('enterhelp')"
                    ></el-input>
                </el-form-item>
            </div>

            <div v-show="hasWalletCom && form.decryptType==='4'">
                <el-form-item class="el-wallet-style mt-40" :label="$t('selectadocument')">
                    <input id="fileUpload2" type="file" style="display:none" @change="uploadFile2">
                    <el-button class="choose-file-btn" @click="selectFile2">{{$t('Selectwalletfile')}}</el-button>
                </el-form-item>

                <el-form-item class="el-wallet-style mt-40" :label="$t('enterPwd')">
                    <el-input class="el-wallet-input"  v-model="form.pwd" type="password"  :placeholder="$t('enterWalletPwd')" auto-complete="off"></el-input>
                </el-form-item>
            </div>
            <!--<div v-show="hasWalletCom && form.decryptType==='4'">-->
            <!--<el-form-item class="el-wallet-style mt-40" label="账户地址">-->
            <!--<el-input class="el-wallet-input"-->
            <!--v-model="form.address"-->
            <!--clearable-->
            <!--&gt;</el-input>-->
            <!--</el-form-item>-->
            <!--</div>-->
        </el-form>
    </div>
</template>

<script>

    export default {
        name: "unlockAccount",
        props: ['hasWallet'],
        data() {
            return {
                form: {
                    decryptType: '4',   //解锁方式,
                    privateKey: '',
                    mnemonic: '',
                    address: '',
                    pwd: '',
                    keystore: {},
                    //fileName: '请选择keystore钱包文件',
                    fileContent: ''
                }
            }
        },
        methods: {
            selectDecryptType(type) {
                this.form.decryptType = type
            },
            importWallet() {
                return new Promise((resolve, reject) => {
                    if (this.form.decryptType === '1') {          //私钥
                        try {
                            if (this.form.privateKey && this.form.privateKey.indexOf('0x') === -1) this.form.privateKey = '0x' + this.form.privateKey
                            if (this.form.privateKey.length !== 66) {
                                this.$message({
                                    message: this.$msg.invalidPrivateKey,
                                    type: 'error'
                                })
                                reject(false)
                                return
                            }
                            resolve(this.form.privateKey)
                            this.form.privateKey = ''
                        } catch (err) {
                            console.log(err);
                            this.$message({
                                message: this.$msg.invalidPrivateKey,
                                type: 'error'
                            })
                            reject(false)
                        }
                    }
                    else if (this.form.decryptType === '2') {          //keystore文件+密码

                        if (!this.form.fileContent) {
                            this.$message({
                                message: this.$msg.selectAnAccFile,
                                type: 'error'
                            })
                            reject(false)
                            return
                        }
                        if (!this.form.pwd) {
                            this.$message({
                                message: this.$msg.enterPwd,
                                type: 'error'
                            })
                            reject(false)
                            return
                        }
                        try {
                            let promise = this.$Wallet.fromEncryptedWallet(this.form.fileContent, this.form.pwd)
                            this.$store.commit('setCryptPercent', {
                                    percent: true,
                                    text: this.$t('inUnlockingAccount')
                                }
                            )
                            promise.then((wallet) => {
                                this.$store.commit('setCryptPercent', {
                                        percent: false,
                                        text: ''
                                    }
                                )
                                resolve(wallet.privateKey)
                                this.form.pwd = ''
                            }, (err) => {
                                this.form.pwd = ''
                                this.$message({
                                    message: this.$msg.unlockFailByPwd,
                                    type: 'error'
                                })
                                this.$store.commit('setCryptPercent', {
                                        percent: false,
                                        text: ''
                                    }
                                )
                                reject(false)
                            })
                        } catch (err) {

                            this.form.fileContent = ''
                            this.form.pwd = ''

                            this.$message({
                                message: this.$msg.unlockFailByPwd,
                                type: 'error'
                            })
                            reject(false)
                        }

                    }
                    else if (this.form.decryptType === '3') {          //助记词
                        try {
                            let wallet = this.$Wallet.fromMnemonic(this.form.mnemonic)
                            resolve(wallet.privateKey)
                            this.form.mnemonic = ''
                        } catch (err) {
                            this.$message({
                                message: this.$msg.invalidMnemonic,
                                type: 'error'
                            })
                            reject(false)
                        }
                    }
                    else if (this.form.decryptType === '4') {          //通过钱包文件导入钱包

                        if (!this.form.fileContent) {
                            this.$message({
                                message: this.$msg.selectAnWalletFile,
                                type: 'error'
                            })
                            reject(false)
                            return
                        }
                        if (!this.form.pwd) {
                            this.$message({
                                message: this.$msg.enterPwd,
                                type: 'error'
                            })
                            reject(false)
                            return
                        }
                        this.$store.commit('setCryptPercent', {
                                percent: true,
                                text: this.$t('importingaccountpleasewait')
                            }
                        )
                        setTimeout(() => {
                            try {
                                let wallet = this.$web3.eth.accounts.wallet
                                wallet.decrypt(this.form.fileContent, this.form.pwd)
                                wallet.myPwd = this.form.pwd
                                wallet.save(this.form.pwd)
                                this.$funs.setActiveAccount(wallet[0].address || wallet[1].address || wallet[2].address || wallet[3].address || wallet[4].address)
                                this.$store.commit('setCryptPercent', {
                                        percent: false,
                                        text: ''
                                    }
                                )
                                resolve()
                            } catch (err) {
                                this.form.pwd = ''
                                this.$store.commit('setCryptPercent', {
                                        percent: false,
                                        text: ''
                                    }
                                )
                                this.$message({
                                    message: this.$msg.unlockFailByPwd,
                                    type: 'error'
                                })
                                reject(false)
                            }
                        }, 430)
                    }
                })
            },
            selectFile1() {
                document.getElementById('fileUpload1').click()
            },
            uploadFile1() {
                let file = document.getElementById('fileUpload1').files[0]
                let reader = new FileReader()
                this.form.fileContent = ''
                this.form.fileName = ''
                this.form.pwd = ''
                reader.readAsText(file)

                reader.onload = () => {
                    let keystore
                    try {
                        keystore = JSON.parse(reader.result)

                        if (keystore.Crypto != null || keystore.crypto != null || (keystore.hash != null && keystore.locked)) {
                            this.form.fileContent = reader.result
                            this.$message({
                                message: this.$msg.corretAccFile,
                                type: 'success'
                            })
                        } else {
                            this.$message({
                                message: this.$msg.invalidAccFile,
                                type: 'error'
                            })
                        }
                    } catch (err) {
                        this.$message({
                            message: this.$msg.invalidFile,
                            type: 'error'
                        })
                    }
                }

                reader.onerror = () => {
                    this.$message({
                        message: this.$msg.readFileErr,
                        type: 'error'
                    })
                }
            },
            selectFile2() {
                document.getElementById('fileUpload2').click()
            },
            uploadFile2() {
                let file = document.getElementById('fileUpload2').files[0]
                let reader = new FileReader()
                this.form.fileContent = ''
                this.form.fileName = ''
                this.form.pwd = ''
                reader.readAsText(file)
                reader.onload = () => {
                    let keystore
                    try {
                        keystore = JSON.parse(reader.result)
                        if (keystore[0].Crypto != null || keystore[0].crypto != null || (keystore[0].hash != null && keystore[0].locked)) {
                            this.form.fileContent = keystore
                            this.$message({
                                message: this.$msg.corretWalletFile,
                                type: 'success'
                            })
                        } else {
                            this.$message({
                                message: this.$msg.invalidWalletFile,
                                type: 'error'
                            })
                        }
                    } catch (err) {
                        this.$message({
                            message: this.$msg.invalidFile,
                            type: 'error'
                        })
                    }
                }

                reader.onerror = () => {
                    this.$message({
                        message: this.$msg.readFileErr,
                        type: 'error'
                    })
                }
            }
        },
        computed: {
            hasWalletCom() {
                if (this.hasWallet === 'hasWallet') {
                    return false
                } else {
                    return true
                }

            },
            decryptInfo() {
                switch (this.form.decryptType) {
                    case '1':
                        return this.$t('importaccounttowalletusingprivatekey')
                    case '2':
                        return this.$t('useyourkeystoragefileUTCtoimportaccounttowallet')
                    case '3':
                        return this.$t('importaccounttowalletusingmnemonicphrases')
                    case '4':
                        return this.$t('importanexistingwallet')
                }
            }
        },
        beforeMount() {          //设置默认的方式
            if (this.hasWallet === 'hasWallet') {
                this.form.decryptType = '1'
            } else {
                this.form.decryptType = '4'
            }

        }
    }
</script>

<style scoped lang="scss" type="text/scss">

    .el-form-item {
        margin-bottom: 0;
    }

    .wallet-decrypt-type {
        .wallet-decrypt-item {
            cursor: default;
            font-size: 16px;
            color: #a2b4db;
            line-height: 22px;
            border-radius: 1px;
            border: solid 1px #465679;
            padding: 5px 15px;
            margin-right: 8px;

            .wallet-decrypt-type-icon {
                display: inline-block;
                width: 22px;
                height: 22px;
                position: relative;
                top: 5px;
            }
            .icon1 {
                background: url("../../assets/images/firstScreen/icon_keys.png");
            }
            .icon1-active {
                background: url("../../assets/images/firstScreen/icon_keys_xz.png");
            }
            .icon2 {
                background: url("../../assets/images/firstScreen/icon_file.png");
            }
            .icon2-active {
                background: url("../../assets/images/firstScreen/icon_file_xz.png");
            }
            .icon3 {
                background: url("../../assets/images/firstScreen/icon_words.png");
            }
            .icon3-active {
                background: url("../../assets/images/firstScreen/icon_words_xz.png");
            }
        }
        .active {
            color: #f39eff;
            border-style: solid;
            border-width: 1px;
            border-image-source: linear-gradient(-16deg,
                #3410f7 0%,
                #711bdc 59%,
                #ad25c0 100%);
            border-image-slice: 1;
        }
    }

    .el-wallet-style p {
        font-size: 16px;
        color: #9388d3;
        line-height: 20px;
        margin-top: 10px;
    }

    .choose-file-btn {
        width: 100%;
        height: 44px;
        background-color: #2a9ce7;
        border-color: #2a9ce7;
        border-radius: 2px;
        font-size: 18px;
        color: #ffffff;
        position: relative;
        left: -0px;
    }

    .fileUploadTip {
        color: #93D3C4
    }
</style>
