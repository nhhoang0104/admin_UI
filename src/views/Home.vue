<template>
  <div class="home">
    <div class="cus-radio-group">
      <div class="header">Tạo dữ liệu cho hệ thống showoff:</div>
      <div class="label">Chọn loại:</div>
      <DxRadioGroup
        :items="dataSrcRadio"
        v-model:value="mode"
        value-expr="value"
        display-expr="label"
        layout="horizontal"
      />
    </div>
    <div class="container">
      <div>
        <div class="file-uploader-block" v-show="!mode">
          <div>Tải dữ liệu hệ thống đã có:</div>
          <DxFileUploader
            select-button-text="Tải file"
            label-text="Chỉ cho phép file: *.json, *.trealet."
            :allowed-file-extensions="['.json', '.trealet']"
            upload-mode="useForm"
            @value-changed="readFileJson"
          />
        </div>

        <div class="flex data__container">
          <div class="left">
            <div><strong>Dữ liệu:</strong></div>
            <div class="dx-fieldset">
              <div class="dx-field">
                <div class="dx-field-label">Tiêu đề web:</div>
                <div class="dx-field-value">
                  <DxTextBox :max-length="50" v-model:value="title" />
                </div>
              </div>
            </div>
            <div class="dx-fieldset">
              <div class="dx-field">
                <div class="dx-field-label">Tiêu đề:</div>
                <div class="dx-field-value">
                  <DxTextBox :max-length="50" v-model:value="txtHeader" />
                </div>
              </div>
            </div>
            <div class="dx-fieldset">
              <div class="dx-field">
                <div class="dx-field-label">Ảnh tiêu đề:</div>
                <div class="dx-field-value">
                  <DxTextBox :max-length="50" v-model:value="imgHeader" />
                </div>
              </div>
            </div>
            <div class="dx-fieldset">
              <div class="dx-field">
                <div class="dx-field-label">Text nút:</div>
                <div class="dx-field-value">
                  <DxTextBox :max-length="50" v-model:value="txtBtnHeader" />
                </div>
              </div>
            </div>
            <div class="dx-fieldset">
              <div class="dx-field">
                <div class="dx-field-label">Tiêu đề các loại:</div>
                <div class="dx-field-value">
                  <DxTextBox
                    :max-length="50"
                    v-model:value="txtHeaderCategory"
                  />
                </div>
              </div>
            </div>
            <div class="tool">
              <div class="add">
                <div class="title">Thêm mới loại:</div>

                <DxButton
                  :width="100"
                  text="Thêm"
                  type="normal"
                  styling-mode="outlined"
                  @click="addItem"
                />
              </div>

              <DxButton
                :width="200"
                text="Xuất file dữ liệu"
                type="normal"
                styling-mode="outlined"
                @click="exportFile"
              />
            </div>
          </div>
          <div class="right">
            <div><strong>Danh sách các loại:</strong></div>

            <div
              class="list"
              v-if="dataForm && dataForm.trealet && dataForm.trealet.items"
            >
              <br />
              <div v-for="(item, index) in dataForm.trealet.items" :key="index">
                <div class="flex flex-v-center flex-gap-10">
                  <div>
                    &nbsp;&nbsp;&nbsp; {{ index + 1 }}. {{ item.title }}:
                  </div>
                  <DxButton
                    :width="100"
                    text="Sửa"
                    type="normal"
                    styling-mode="outlined"
                    @click="updateItem(index)"
                  />
                  <DxButton
                    :width="100"
                    text="Xóa"
                    type="normal"
                    styling-mode="outlined"
                    @click="removeItem(index)"
                  />
                </div>
                <br />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <FormData
      :isShowed="isShowForm"
      :dataSrc="categorySelected"
      :indexItem="indexCategorySelected"
      @close="closeFormData"
      @save="addNewCategory"
    />
  </div>
</template>

<script>
import FormData from "@/views/FormData.vue";
import { DxFileUploader } from "devextreme-vue/file-uploader";
import DxButton from "devextreme-vue/button";
import DxRadioGroup from "devextreme-vue/radio-group";
import DxTextBox from "devextreme-vue/text-box";
import _ from "lodash";

export default {
  components: {
    FormData,
    DxFileUploader,
    DxButton,
    DxRadioGroup,
    DxTextBox,
  },

  data() {
    return {
      isShowForm: false,
      title: "",
      txtHeader: "",
      imgHeader: "",
      txtBtnHeader: "",
      dataSrcRadio: [
        { label: "Dữ liệu có sẵn", value: 0 },
        { label: "Tạo mới dữ liệu", value: 1 },
      ],
      modelForm: {
        trealet: {
          title: "",
          txtHeader: "",
          imgHeader: "",
          txtBtnHeader: "",
          txtHeaderCategory: "",
          exec: "streamline",
          items: [],
        },
      },
      dataForm: null,
      categorySelected: null,
      indexCategorySelected: -1,
      mode: 0,
      labelBtn: "Thêm",
    };
  },

  watch: {
    mode(newVal) {
      if (newVal === 1) {
        this.dataForm = _.cloneDeep(this.modelForm);
      }
    },
  },

  methods: {
    // mở form
    openFormData() {
      this.isShowForm = true;
    },

    // đóng form
    closeFormData() {
      this.isShowForm = false;
    },

    // thêm category
    addNewCategory(category = null, index) {
      this.isShowForm = false;
      if (category === null) return;

      if (this.dataForm) {
        if (index === -1) {
          this.dataForm.trealet.items.push(category);
          return;
        } else {
          this.dataForm.trealet.items[index] = _.cloneDeep(category);
        }
      }
    },

    // xử lý đọc file
    readFileJson({ value }) {
      let me = this;
      if (value && value[0]) {
        let file = value[0];
        let fr = new FileReader();

        fr.onload = function(e) {
          try {
            let temp = JSON.parse(e.target.result);
            if (temp && temp.trealet) {
              me.dataForm = JSON.parse(e.target.result);
              me.title = _.cloneDeep(me.dataForm.trealet.title);
              me.txtHeader = _.cloneDeep(me.dataForm.trealet.txtHeader);
              me.imgHeader = _.cloneDeep(me.dataForm.trealet.imgHeader);
              me.txtBtnHeader = _.cloneDeep(me.dataForm.trealet.txtBtnHeader);
              me.txtHeaderCategory = _.cloneDeep(
                me.dataForm.trealet.txtHeaderCategory
              );
            }
          } catch (error) {
            me.dataForm = _.cloneDeep(me.modelForm);
          }
        };

        fr.readAsText(file);
      }
    },

    // xuất file
    exportFile() {
      let data = null;

      if (!this.dataForm) {
        this.dataForm = _.cloneDeep(this.modelForm);
      }

      this.dataForm.trealet.title = this.title;
      this.dataForm.trealet.txtHeader = this.txtHeader;
      this.dataForm.trealet.imgHeader = this.imgHeader;
      this.dataForm.trealet.txtBtnHeader = this.txtBtnHeader;
      this.dataForm.trealet.txtHeaderCategory = this.txtHeaderCategory;

      data = JSON.stringify(this.dataForm);

      const blob = new Blob([data], { type: "application/json" });

      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "streamline.trealet";
      link.click();
      URL.revokeObjectURL(link.href);
    },

    /**
     * add
     */
    addItem() {
      this.categorySelected = null;
      this.indexCategorySelected = -1;
      this.openFormData();
    },

    /**
     * update item
     * @params index
     * @returns
     */
    updateItem(index) {
      this.categorySelected = _.cloneDeep(this.dataForm.trealet.items[index]);
      this.indexCategorySelected = index;
      this.openFormData();
    },

    /**
     * remove item
     */
    removeItem(index) {
      if (index > -1) {
        this.dataForm.trealet.items.splice(index, 1);
      }
    },
  },
};
</script>

<style lang="scss">
.home {
  width: 900px;
  margin: 0 auto;
  border-radius: 5px;
  padding: 10px;
  box-sizing: border-box;
  background: white;
  box-shadow: 5px white;
}

.cus-radio-group {
  margin: 20px;
  background: white;
  .header {
    padding: 10px 0px;
    font-weight: bold;
    font-size: 36px;
  }

  .label {
    padding: 10px 0px;
    font-weight: bold;
  }
}

.container {
  margin-left: 20px;
  background-color: white !important;
  .dx-fileuploader-wrapper {
    padding: 7px 0px;
  }

  .tool {
    display: flex;
    flex-direction: column;
    gap: 10px;
    .add {
      display: flex;
      align-items: center;
      gap: 10px;
      height: 50px;

      .title {
        font-weight: bold;
        margin: 20px 0px;
      }
    }
  }
}

.flex {
  display: flex !important;
}

.flex-v-center {
  align-items: center;
}

.flex-gap-10 {
  gap: 10px;
}

.data__container {
  .left,
  .right {
    width: 50%;
  }
}
</style>
