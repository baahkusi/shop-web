<template>
  <el-card v-if="hasProfile">
    <el-form :model="accountInfo" ref="accountInfo" label-position="top">
      <el-form-item label="Your Handle" prop="name" required>
        <el-input v-model="accountInfo.name"></el-input>
      </el-form-item>
      <el-row :gutter="20">
        <el-col :span="12">
          <el-form-item label="Phone" prop="phone" required>
            <el-input v-model="accountInfo.phone">
              <el-form-item class="code" prop="code" slot="prepend" required>
                <el-select v-model="accountInfo.code" style="width:120px" filterable>
                  <el-option
                    v-for="code in dialCode"
                    :key="code.code"
                    :label="code.code +' '+ code.dial_code"
                    :value="code.dial_code"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label=".">
            <el-button
              @click="activatePhone"
              type="info"
              :disabled="phone_verified"
            >{{ activeTextPhone }}</el-button>
            <el-input
              v-model="pcode"
              placeholder="Enter pin sent to your phone."
              :disabled="phone_verified"
              v-if="activePhone"
            ></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row :gutter="20">
        <el-col :span="12">
          <el-form-item label="Your Email" prop="email" required>
            <el-input v-model="accountInfo.email"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label=".">
            <el-button
              @click="activateEmail"
              type="info"
              :disabled="email_verified"
            >{{ activeTextEmail }}</el-button>
            <el-input
              v-model="ecode"
              placeholder="Enter pin sent to your email."
              :disabled="email_verified"
              v-if="activeEmail"
            ></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="Your Status" prop="level">
        <el-input v-model="accountInfo.level" disabled></el-input>
      </el-form-item>
    </el-form>
    <el-button type="primary" @click="save" size="small" style="width:100%;margin-top:15px">Save</el-button>
  </el-card>
</template>
<script>
import dialCode from "../../../data/dialCode.js";
export default {
  props: ["profile", "hasProfile", "isSeller"],
  data() {
    return {
      email_verified: false,
      activeEmail: false,
      activeTextEmail: "Verify Email.",
      phone_verified: false,
      activePhone: false,
      activeTextPhone: "Verify Phone.",
      ecode: "",
      pcode: "",
      accountInfo: {
        name: "",
        email: "",
        level: "",
        code: "",
        phone: ""
      }
    };
  },
  computed: {
    dialCode() {
      return dialCode;
    }
  },
  methods: {
    async verifyAccount(phase, medium, code = 0) {
      const payload = {
        "111": {
          activate_account: {
            phase: phase,
            medium: medium,
            code: code
          },
          "000": ["activate_account"]
        },
        "000": ["111"]
      };
      const fp = await this.$browser();
      let verify = new Promise((resolve, reject) => {
        this.$actions
          .post("/action", payload, {
            headers: { 
            Authorization: this.profile.token, 
            'Account-ID':this.profile.user.email, 
            'Device-ID': fp.deviceHash
            }
          })
          .then(response => {
            // handle success
            const resp = response.data["111"].activate_account;

            
            if (resp.status) {
              this.$notify.success({
                title: "Success",
                message: resp.msg
              });
              resolve(response);
            } else {
              this.$notify.error({
                title: "Error",
                message: resp.msg
              });
              reject(response);
            }
          })
          .catch(error => {
            // handle error
            reject();
            this.$notify.error({
              title: "Error",
              message: "Update Error."
            });
          });
      });
      return verify;
    },
    activateEmail() {
      if (this.activeEmail) {
        if (this.ecode) {
          this.verifyAccount("activate", "email", this.ecode).then(res => {
            this.activeEmail = false;
            this.activeTextEmail = "Email Activated.";
            this.email_verified = true;
            const msg = "Email verified.";
            this.$notify.success({
              title: "Success",
              message: msg
            });
            return;
          });
        }
        const error = "Please provide activation code.";
        this.$notify.error({
          title: "Error",
          message: error
        });
        return;
      }
      this.verifyAccount("generate", "email").then(res => {
        this.activeEmail = true;
        this.activeTextEmail = "Send Code.";
      });
    },
    activatePhone() {
      if (this.activePhone) {
        if (this.pcode) {
          this.verifyAccount("activate", "phone", this.pcode).then(res => {
            this.activePhone = false;
            this.activeTextPhone = "Phone Activated.";
            this.phone_verified = true;
            const msg = "Phone verified.";
            this.$notify.success({
              title: "Success",
              message: msg
            });
            return;
          });
        }
        const error = "Please provide activation code.";
        this.$notify.error({
          title: "Error",
          message: error
        });
        return;
      }
      this.verifyAccount("generate", "phone").then(res => {
        this.activePhone = true;
        this.activeTextPhone = "Send Code.";
      });
    },
    save() {
      let isValid;
      this.$refs.accountInfo.validate(valid => (isValid = valid));

      if (isValid) {
        const accountInfo = {
          name: this.accountInfo.name,
          email: this.accountInfo.email,
          phone: this.accountInfo.code + " " + this.accountInfo.phone
        };
        const params = { info: "account", data: accountInfo };
        this.$eventBus.$emit("save-profile-settings", params);
      } else {
        const error = "Please provide the required information.";
        this.$notify.error({
          title: "Error",
          message: error
        });
        return;
      }
    }
  },
  created() {
    if (this.hasProfile) {
      let phone_code;
      if (this.profile.user.phone) {
        phone_code = this.profile.user.phone.split(" ");
      } else {
        phone_code = ["", ""];
      }

      this.accountInfo = {
        name: this.profile.user.name,
        email: this.profile.user.email,
        level: this.profile.user.level,
        code: phone_code[0],
        phone: phone_code[1]
      };
      this.phone_verified = this.profile.user.phone_verified;
      this.email_verified = this.profile.user.email_verified;
    }
  }
};
</script>

<style scoped>
.el-form-item.code {
  margin-bottom: 0px;
}
</style>