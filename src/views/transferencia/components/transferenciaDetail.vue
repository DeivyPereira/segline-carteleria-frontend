<template>
  <div class="createPost-container">
    <el-form class="form-container" :model="postForm" :rules="rules" ref="postForm">
      <div class="createPost-main-container">
        <el-row>
          <div class="postInfo-container">
            <el-row class="animated fadeIn">
              <el-col :span="4" class="animated fadeIn" style="padding: 10px 0px;"></el-col>
              <el-col :span="6" class="animated fadeIn" style="padding: 10px 0px;">
               <button> <img :src="plantilla1" />
                <br />Plantilla 1</button>
              </el-col>
              <el-col :span="6" class="animated fadeIn" style="padding: 10px 0px;">
                <button>   <img :src="plantilla2" />
                <br />Plantilla 2</button>
              </el-col>
              <el-col :span="6" class="animated fadeIn" style="padding: 10px 0px;">
                 <button>  <img :src="plantilla3" />
                <br />Plantilla 3</button>
              </el-col>
            </el-row>
            <el-row class="animated fadeIn">
              <el-col :span="12">
                <el-form-item style="margin-bottom: 40px;" label="Sede:">
                  <br />
                  <el-select placeholder="Seleccionar" v-model="postForm.sede">
                    <el-option
                      v-for="item in sedes"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item style="margin-bottom: 40px;" label="Televisor:">
                  <br />
                  <el-select placeholder="Seleccionar" v-model="postForm.tv">
                    <el-option
                      v-for="item in tv"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row class="animated fadeIn">
              <el-col :span="24" class="animated fadeIn" style="padding: 10px 0px;">
                <dropzone
                  v-on:dropzone-success="dropzoneS"
                  v-on:dropzone-removedFile="dropzoneR"
                  v-on:dropzone-processing="dropzoneP"
                  v-on:dropzone-error="dropzoneE"
                  id="myVueDropzone"
                  :url="urlupload"
                ></dropzone>
              </el-col>
            </el-row>
          </div>
        </el-row>
      </div>
    </el-form>
  </div>
</template>
<script>
import plantilla1 from "@/assets/imgs/1tv.png";
import plantilla2 from "@/assets/imgs/2tv.png";
import plantilla3 from "@/assets/imgs/3tv.png";

import { create, update, get, getAll } from "@/api/apigeneral";
import { filterItems, filterValue } from "@/filters";
import MDinput from "@/components/MDinput";
import Multiselect from "vue-multiselect";
import "vue-multiselect/dist/vue-multiselect.min.css";
import { validateEmail, validateNumber } from "@/utils/validate";
import waves from "@/directive/waves";
import Tinymce from "@/components/Tinymce";
import Upload from "@/components/Upload/singleImage3";
import Dropzone from "@/components/Dropzone";

import Vue from "vue";

const transactionForm = {
  id: undefined,
  paisReceive: "Venezuela",
  amountReceive: "",
  typeMoney: "",
  dateOperation: "",
  voucherReceive: "",
  numOperation: "",
  paisSend: "",
  amountSend: "",
  voucherSend: "",
  commission: "",
  message: "",
  repayment: "",
  typePayment: "",
  status: 1,
  clientId: undefined,
  account_bankId: undefined,
  userApprovedId: undefined,
  bankId: undefined,
  gestorId: undefined,
  tax: undefined,
  sede: undefined,
  tv: undefined,
  sedes: undefined,
  Televisor: undefined
};
const clienteForm = {
  id: undefined,
  name: "",
  lastname: "",
  email: "",
  dni: "",
  phone: "",
  status: "",
  nationality: "",
  phone2: ""
};
const taxInfo = {
  id: undefined,
  pais: "",
  money: "",
  date: "",
  tax: ""
};
const accountBank = {
  id: undefined,
  name: "",
  lastname: "",
  typeDocument: "",
  dni: "",
  email: "",
  phone: "",
  typeAccount: "",
  accountNumber: "",
  bank: "",
  pais: "Venezuela",
  status: 1,
  alias: "",
  clientId: ""
};
export default {
  name: "transferenciaDetail",
  components: { MDinput, Multiselect, Tinymce, Upload, Dropzone },
  props: {
    isEdit: {
      type: Boolean,
      default: false
    }
  },
  directives: {
    waves
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      if (value === "") {
        callback(new Error(rule.name + " es requerido"));
      } else {
        callback();
      }
    };
    return {
      //Configuracion
      configuration: undefined,
      gestor: undefined,
      plantilla1,
      plantilla2,
      plantilla3,
      //Fin Configuracion
      imagenval: 0,
      imagen: [],
      urlupload: process.env.BASE_API + "multimedia",
      markers: [],
      places: [],
      currentPlace: null,
      postForm: Object.assign({}, transactionForm),
      postFormCliente: Object.assign({}, clienteForm),
      postFormBank: Object.assign({}, accountBank),
      urlimprimir: process.env.BASE_API + "imprimir/",
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 1000,
        codigo: undefined,
        sort: "DESC"
      },
      loading: false,
      pross: 0,
      status: null,
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "Editar",
        create: "Crear"
      },
      dialogBankVisible: false,
      dialogStatusBank: "",
      dialogPvVisible: false,
      bankAccountAccept: false,
      rules: {
        clientId: [{ validator: validateRequire, name: "Cliente" }],
        account_bankId: [
          { validator: validateRequire, name: "Banco del Cliente" }
        ],
        paisReceive: [{ validator: validateRequire, name: "Pais que recibe" }],
        amountReceive: [{ validator: validateRequire, name: "Monto" }],
        typeMoney: [{ validator: validateRequire, name: "Tipo de Moneda" }],
        dateOperation: [
          { validator: validateRequire, name: "Fecha de Operación" }
        ],
        paisSend: [{ validator: validateRequire, name: "Pais que Envia" }],
        amountSend: [{ validator: validateRequire, name: "Monto que envia" }],
        bankId: [{ validator: validateRequire, name: "Banco Receptor" }],
        status: [{ validator: validateRequire, name: "Estado" }],
        typePayment: [{ validator: validateRequire, name: "Tipo de pago" }],
        tax: [{ validator: validateRequire, name: "Tasa" }]
      },
      rulesclient: {
        name: [{ validator: validateRequire, name: "Nombre" }],
        lastname: [{ validator: validateRequire, name: "Apellidos" }],
        email: [{ validator: validateRequire, name: "Email / Correo" }],
        dni: [{ validator: validateRequire, name: "Cedula / Dni" }],
        phone: [{ validator: validateRequire, name: "Telefono" }],
        status: [{ validator: validateRequire, name: "Estado" }]
      },
      rulesaccountbank: {
        name: [{ validator: validateRequire, name: "Nombre" }],
        lastname: [{ validator: validateRequire, name: "Apellidos" }],
        typeDocument: [
          { validator: validateRequire, name: "Tipo de documento" }
        ],
        dni: [{ validator: validateRequire, name: "Cedula" }],
        typeAccount: [{ validator: validateRequire, name: "Tipo de Cuenta" }],
        accountNumber: [
          { validator: validateRequire, name: "Numero de cuenta" }
        ],
        bank: [{ validator: validateRequire, name: "Banco" }],
        clientId: [{ validator: validateRequire, name: "cliente" }]
      },

      transacionDisabled: false,
      // Caracteristicas
      checkAll: false,
      checkedLoading: false,
      isIndeterminate: true,
      // Tasas
      paistaxes: [],
      moneytaxes: [],
      taxId: undefined,
      datetax: "",
      //Bancos receptores
      accountbanks: [],
      // Depositos
      depositAlert: false,
      depositType: undefined,
      depositMessage: "",
      sedes: [
        {
          value: "Sede 1",
          label: "Sede 1"
        },
        {
          value: "Sede 2",
          label: "Sede 2"
        },
        {
          value: "Sede 3",
          label: "Sede 3"
        },
        {
          value: "Sede 4",
          label: "Sede 4"
        }
      ],
      tv: [
        {
          value: "Tv 1",
          label: "Tv 1"
        },
        {
          value: "Tv 2",
          label: "Tv 2"
        },
        {
          value: "Tv 3",
          label: "Tv 3"
        },
        {
          value: "Tv 4",
          label: "Tv 4"
        }
      ]
    };
  },
  created() {},
  methods: {
    dropzoneS(file, xhr, response) {
      this.postForm.voucherReceive = file.xhr.response;
      console.log("Success de simple");
      this.$message({ message: "Imagen agregada", type: "success" });
    },
    dropzoneR(file) {
      console.log("Remove File");
    },
    dropzoneE(file, xhr, response) {
      console.log("Error File");
    },
    dropzoneP(file, xhr, response) {
      console.log("Processing");
    },

    SaveTransaction() {
      this.$refs.postForm.validate(valid => {
        if (valid) {
          this.loading = true;
          this.transacionDisabled = true;
          if (this.isEdit) {
            update(this.postForm, "transaction", this.postForm.id)
              .then(response => {
                this.$message({
                  title: "Éxito",
                  message: "Transacción actualizado",
                  type: "success",
                  duration: 2000
                });
                setTimeout(
                  () => this.$router.push({ path: "/transferencia" }),
                  2000
                );
              })
              .catch(error => {
                console.log(error);
                this.loading = false;
                this.transacionDisabled = false;
              });
          } else {
            create(this.postForm, "transaction")
              .then(response => {
                this.$message({
                  title: "Éxito",
                  message: "Transacción creada",
                  type: "success",
                  duration: 2000
                });
                window.open(this.urlimprimir + response.response, "_blank");
                setTimeout(
                  () => this.$router.push({ path: "/transferencia" }),
                  2000
                );
              })
              .catch(error => {
                console.log(error);
                this.loading = false;
                this.transacionDisabled = false;
              });
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    handleSizeChange(val) {
      this.listQuery.limit = val;
    },
    handleCurrentChange(val) {
      this.listQuery.page = val;
    }
  }
};
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "src/styles/mixin.scss";

.avatar {
  width: 200px;
  height: 200px;
  border-radius: 50%;
}
.createPost-container {
  position: relative;
  .createPost-main-container {
    padding: 40px 45px 20px 50px;
    .postInfo-container {
      position: relative;
      @include clearfix;
      margin-bottom: 10px;
      .postInfo-container-item {
        float: left;
      }
    }
    .editor-container {
      min-height: 500px;
      margin: 0 0 30px;
      .editor-upload-btn-container {
        text-align: right;
        margin-right: 10px;
        .editor-upload-btn {
          display: inline-block;
        }
      }
    }
  }
  .word-counter {
    width: 40px;
    position: absolute;
    right: -10px;
    top: 0px;
  }
}
.panel-group {
  margin-top: 18px;
  .card-panel-col {
    margin-bottom: 32px;
  }
  .card-panel {
    height: 108px;
    cursor: pointer;
    font-size: 12px;
    position: relative;
    overflow: hidden;
    color: #666;
    background: #fff;
    box-shadow: 4px 4px 40px rgba(0, 0, 0, 0.05);
    border-color: rgba(0, 0, 0, 0.05);
    &:hover {
      .card-panel-icon-wrapper {
        color: #fff;
      }
      .icon-people {
        background: #40c9c6;
      }
      .icon-message {
        background: #36a3f7;
      }
      .icon-money {
        background: #f4516c;
      }
      .icon-shoppingCard {
        background: #34bfa3;
      }
    }
    .icon-people {
      color: #40c9c6;
    }
    .icon-message {
      color: #36a3f7;
    }
    .icon-money {
      color: #f4516c;
    }
    .icon-shoppingCard {
      color: #34bfa3;
    }
    .card-panel-icon-wrapper {
      float: left;
      margin: 14px 0 0 14px;
      padding: 16px;
      transition: all 0.38s ease-out;
      border-radius: 6px;
    }
    .card-panel-icon {
      float: left;
      font-size: 48px;
    }
    .card-panel-description {
      float: right;
      font-weight: bold;
      margin: 26px;
      margin-left: 0px;
      .card-panel-text {
        line-height: 18px;
        color: rgba(0, 0, 0, 0.45);
        font-size: 16px;
        margin-bottom: 12px;
      }
      .card-panel-num {
        font-size: 20px;
      }
    }
    .card-panel-description-center {
      text-align: center;
      font-weight: bold;
      margin: 26px;
      margin-left: 0px;
      .card-panel-text {
        line-height: 18px;
        color: rgba(0, 0, 0, 0.45);
        font-size: 16px;
        margin-bottom: 12px;
      }
      .card-panel-num {
        font-size: 20px;
      }
    }
  }
}
.grid-container {
  display: grid;
  grid-template-columns: auto auto auto auto;
  grid-gap: 10px;
  padding: 10px;
  border-radius: 5px;
}
.grid-container > div {
  background-color: #e7eff7;
  text-align: center;
  padding: 20px 0;
  font-size: 30px;
}
.bg-danger {
  border: solid 4px #fa6b18 !important;
  border-radius: 8px;
}
/* Container needed to position the button. Adjust the width as needed */
.imgcontainer {
  position: relative;
  width: 100%;
}

/* Make the image responsive */
.imgcontainer img {
  width: 100%;
  height: auto;
}

/* Style the button and place it in the middle of the container/image */
.imgcontainer .btn {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  -ms-transform: translate(-50%, -50%);
  background-color: #555;
  color: white;
  font-size: 16px;
  padding: 12px 24px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.imgcontainer .btn:hover {
  background-color: black;
}

.animated {
  -webkit-animation-duration: 1s;
  animation-duration: 1s;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
}

.animated.infinite {
  -webkit-animation-iteration-count: infinite;
  animation-iteration-count: infinite;
}

.animated.hinge {
  -webkit-animation-duration: 2s;
  animation-duration: 2s;
}

.animated.bounceIn,
.animated.bounceOut,
.animated.flipOutX,
.animated.flipOutY {
  -webkit-animation-duration: 0.75s;
  animation-duration: 0.75s;
}

@-webkit-keyframes fadeIn {
  from {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

.fadeIn {
  -webkit-animation-name: fadeIn;
  animation-name: fadeIn;
}
.is-disabled {
  color: #121213 !important;
}
</style>
