<template>
  <div>
    <br />
    <el-alert
      title="Register"
      type="info"
      description="Please provide your info to register."
      :closable="false"
      show-icon
    ></el-alert>
    <br />
    <el-form :model="formModel" ref="formModel" label-width="120px" class="demo-dynamic">
      <el-form-item
        prop="email"
        label="Email"
        :rules="[
      { required: true, message: 'Please input email address', trigger: 'blur' },
      { type: 'email', message: 'Please input correct email address', trigger: 'blur' }
    ]"
      >
        <el-input size="small" v-model="formModel.email"></el-input>
      </el-form-item>
      <el-form-item
        label="Password"
        prop="pass"
        :rules="[
      { required: true, message: 'Password Required.', trigger: 'blur' },
      { min: 6, message: 'Password must be at least 6 characters.', trigger: 'blur' },
      { pattern: /^\S*$/, message: 'No spaces allowed.'}
    ]"
      >
        <el-input size="small" type="password" v-model="formModel.pass" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item
        label="Confirm"
        prop="confirm"
        :rules="[
      { required: true, message: 'Confirm Password Required.', trigger: 'blur' },
      { min: 6, message: 'Password must be at least 6 characters.', trigger: 'blur' },
      { pattern: /^\S*$/, message: 'No spaces allowed.'},
      { validator: confirm, message: 'Passwords don\'t match!' }
    ]"
      >
        <el-input size="small" type="password" v-model="formModel.confirm" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button size="small" type="primary" @click="submitForm('formModel')">Register</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
export default {
  data() {
    return {
      formModel: {
        email: "",
        pass: "",
        confirm: ""
      }
    };
  },
  methods: {
    submit() {
      const loading = this.$loading({
        lock: true,
        text: "Registering...",
        spinner: "el-icon-loading",
        background: "rgba(255, 255, 255, 0.5)"
      });
      const payload = {
        "111": {
          register: {
            email: this.formModel.email.trim(),
            password: this.formModel.pass.trim()
          },
          "000": ["register"]
        },
        "000": ["111"]
      };

      this.$actions
        .post("/action", payload)
        .then(response => {
          // console.log(response);
          const resp = response.data["111"].register;
          if (resp.status) {
            this.$notify.success(resp.msg);
            this.$router.push(`/auth/login`);
          } else{
            this.$notify.error(resp.msg);
          }
        })
        .catch(error => {
          // console.log(error);
        })
        .finally(() => {
          loading.close();
          // always executed
          // remove loader
        });
    },
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.submit();
        } else {
          this.$notify.error("Form Invalid");
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    confirm(rule, value, callback) {
      if (value !== this.formModel.pass) {
        callback(new Error("Passwords don't match!"));
      } else {
        callback();
      }
    }
  }
};
</script>