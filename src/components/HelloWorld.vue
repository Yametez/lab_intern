<template>
  <v-app>
    <v-container>
      <v-row>
        <!-- Form section for entering data -->
        <v-col cols="12" md="4">
          <v-card class="pa-4">
            <v-card-title>{{
              editMode ? "แก้ไขข้อมูล" : "เพิ่มข้อมูล"
            }}</v-card-title>
            <v-card-text>
              <v-form ref="form" v-model="valid">
                <v-text-field
                  v-model="formData.firstName"
                  label="ชื่อ"
                  :rules="[rules.required]"
                ></v-text-field>
                <v-text-field
                  v-model="formData.lastName"
                  label="นามสกุล"
                  :rules="[rules.required]"
                ></v-text-field>
                <v-text-field
                  v-model="formData.email"
                  label="อีเมล"
                  :rules="[rules.required, rules.email]"
                ></v-text-field>
                <v-text-field
                  v-model="formData.phone"
                  label="เบอร์โทร"
                  :rules="[rules.required, rules.phone]"
                ></v-text-field>
                <v-btn
                  color="success"
                  class="mr-2"
                  @click="editMode ? updateItem() : submit()"
                  :disabled="!valid"
                >
                  {{ editMode ? "บันทึกการแก้ไข" : "Submit" }}
                </v-btn>
                <v-btn color="grey" @click="cancelEdit" v-if="editMode">
                  ยกเลิก
                </v-btn>
              </v-form>
            </v-card-text>
          </v-card>
        </v-col>

        <!-- Data table section -->
        <v-col v-if="items && items.length" cols="12" md="8">
          <v-card class="pa-4">
            <v-card-title>รายการข้อมูล</v-card-title>
            <v-card-text>
              <v-data-table  v-if="items.length" :headers="headers" :items="items">
                <template
                 
                  v-slot:[`item.actions`]="{ item }"
                >
                <!-- {{ item }} -->
                  <v-icon small @click="editItem(item)"> mdi-pencil </v-icon>
                  <v-icon small @click="deleteItem(item.id)">
                    mdi-delete
                  </v-icon>
                </template>
              
              </v-data-table>
             
            </v-card-text>
            
          </v-card>
        </v-col>
        <v-col v-else cols="12" md="8">
          <v-card class="pa-4 d-flex justify-center text-center">
            <v-card-text>ไม่มีข้อมูล</v-card-text>
          </v-card>
        </v-col>
      </v-row>

      <!-- Snackbar for success messages -->
      <v-snackbar
        v-model="snackbar"
        :timeout="3000"
        top
        right
        :color="snackbarColor"
      >
        <v-row justify="space-between" align="center">
          <v-col>{{ snackbarMessage }}</v-col>
          <v-col class="d-flex justify-end">
            <v-btn color="white" text @click="snackbar = false">x</v-btn>
          </v-col>
        </v-row>
      </v-snackbar>
    </v-container>
  </v-app>
</template>


<script>
import axios from "axios";

export default {
  data() {
    return {
      valid: false,
      formData: {
        firstName: "",
        lastName: "",
        email: "",
        phone: "",
      },
      DNS_IP: "http://localhost:5001",
      items: [],
      headers: [
        { text: "ชื่อ", value: "firstName" },
        { text: "นามสกุล", value: "lastName" },
        { text: "อีเมล", value: "email" },
        { text: "เบอร์โทร", value: "phone" },
        { text: "", value: "actions", sortable: false },
      ],
      rules: {
        required: (value) => !!value || "กรุณากรอกข้อมูล",
        email: (value) => /.+@.+\..+/.test(value) || "อีเมลต้องถูกต้อง",
        phone: (value) => /^\d+$/.test(value) || "เบอร์โทรต้องเป็นตัวเลข",
      },
      snackbar: false, // State for controlling the snackbar visibility
      snackbarMessage: "", // Message to be displayed in the snackbar
      snackbarColor: "", // Color of the snackbar
      editMode: false, // State to determine if we are in edit mode
      editedIndex: -1, // Index of the item being edited
    };
  },
  mounted() {
    this.getdata();
  },

  methods: {
    async submit() {
      if (this.$refs.form.validate()) {
        // this.items.push({ ...this.formData });
        if (this.formData.id) {
          await this.postedit(this.formData.id);
        } else {
          await this.postdata();
        }
        await this.resetForm();
      }
    },
    //get data
    getdata() {
      axios.get(this.DNS_IP + "/labApi/get").then(async (res) => {
        if (res.data.status === true) {
          this.items = res.data;
        } else {
          this.items = null;
        }
        console.log("test", res);
        this.items = res.data;
      });
    },
    //post data
    postdata() {
      let data1 = {
        firstName: this.formData.firstName,
        lastName: this.formData.lastName,
        email: this.formData.email,
        phone: this.formData.phone,
      };
      console.log(data1);
      axios.post(this.DNS_IP + `/labApi/add`, data1).then(async (res) => {
        await this.showSnackbar("เพิ่มข้อมูลสำเร็จ", "success");
        await this.getdata();
        console.log(res);
      });
    },
    //edit data
    async editItem(item) {
      this.editMode = true;
      this.editedIndex = this.items.indexOf(item);
      this.formData = { ...item };

      console.log("editItem", this.formData);
    },
    //update data
    updateItem() {
      if (this.$refs.form.validate()) {
        Object.assign(this.items[this.editedIndex], this.formData);
        this.resetForm();
        this.showSnackbar("แก้ไขข้อมูลสำเร็จ", "info");
      }
    },
    //post edit
    postedit(id) {
      let data1 = {
        firstName: this.formData.firstName,
        lastName: this.formData.lastName,
        email: this.formData.email,
        phone: this.formData.phone,
      };
      console.log(data1);
      axios
        .post(this.DNS_IP + `/labApi/edit/` + id, data1)
        .then(async (res) => {
          await this.showSnackbar("แก้ไขข้อมูลสำเร็จ", "success");
          console.log(res);
        });
    },
    async deleteItem(id) {
      console.log(id)
      if (confirm(`คุณต้องการลบข้อมูลนี้หรือไม่`)) {
        try {
          
          await axios
            .post(this.DNS_IP + `/labApi/delete/${id}`)
            .then(async (responeData) => {
              await this.getdata();
              console.log(responeData);
            });

          this.showSnackbar("ลบข้อมูลสำเร็จ", "error");
        } catch (error) {
          console.error(error);
          this.showSnackbar("เกิดข้อผิดพลาดในการลบข้อมูล", "error");
        }
      }
    },
    cancelEdit() {
      this.resetForm();
    },
    resetForm() {
      this.editMode = false;
      this.editedIndex = -1;
      this.formData = {
        firstName: "",
        lastName: "",
        email: "",
        phone: "",
      };
      this.$refs.form.resetValidation();
    },
    showSnackbar(message, color) {
      this.snackbarMessage = message;
      this.snackbarColor = color;
      this.snackbar = true;
    },
  },
  //delete data
    
    //end
    
    
};
</script>

<style>
.v-card {
  margin-bottom: 20px;
}
</style>
