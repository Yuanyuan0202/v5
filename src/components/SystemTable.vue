<template>
  <div class="system-variable">
    <div class="div">
      <div class="overlap-group">
        <img class="v" :src="require('@/assets/logo.png')" />
        <div class="frame">
          <div class="div-wrapper"><div class="text-wrapper">DICOM List</div></div>
          <div class="div-wrapper"><div class="text-wrapper">CAD Config</div></div>
          <div class="div-wrapper"><div class="text-wrapper">History</div></div>
          <div class="frame-2"><div class="text-wrapper-2">System</div></div>
        </div>
        <div class="icon"><img class="union" :src="require('@/assets/union.svg')" /></div>
      </div>
      <div class="frame-3">
        <img class="img" :src="require('@/assets/frame-60.svg')" />
        <div class="rectangle"></div>
        <div class="rectangle"></div>
        <div class="rectangle"></div>
        <div class="rectangle-2"></div>
        <div class="rectangle"></div>
        <div class="rectangle"></div>
        <img class="img" :src="require('@/assets/frame-59.svg')" />
      </div>

      <div class="frame-8-title">
        <div class="text-wrapper-title">Name</div>
        <div class="text-wrapper-title">Value</div>
        <div class="text-wrapper-title">Description</div>
        <div class="text-wrapper-title">Edit</div>
      </div>
      <div class="frame-4" id="data-container">
        <div v-for="(item, index) in paginatedData" :key="index" class="frame-8">
          <div class="text-wrapper-5">{{ item.name }}</div>
          <div class="value">
            <div class="frame-9">
              <template v-if="typeof item.value === 'boolean'">
                <div class="toggle-container" @click="toggleBooleanValue(index)">
                  <div :class="['frameParent', { 'true': item.value, 'false': !item.value }]">
                    <div :class="['trueWrapper', { 'active': item.value }]">
                      <div class="true">TRUE</div>
                    </div>
                    <div :class="['falseWrapper', { 'active': !item.value }]">
                      <div class="false">FALSE</div>
                    </div>
                  </div>
                </div>
              </template>
              <span v-else-if="!item.editMode" class="text-wrapper-6">{{ item.value }}</span>
              <input v-if="item.editMode && typeof item.value !== 'boolean'" class="edit-input edit-value-input" type="text" v-model="item.value" />
            </div>
          </div>
          <div class="frame-6">
            <div class="text-wrapper-5">{{ item.description }}</div>
            <div class="div-2">
              <div v-if="!item.editMode" class="edit-icon" @click="toggleEditMode(index)" :style="{ backgroundImage: 'url(' + require('@/assets/edit-icon.svg') + ')' }"></div>
              <div v-if="item.editMode" class="save-icon" @click="saveChanges(index)" :style="{ backgroundImage: 'url(' + require('@/assets/save-icon.svg') + ')' }"></div>
              <div v-if="item.editMode" class="cancel-icon" @click="cancelChanges(index)" :style="{ backgroundImage: 'url(' + require('@/assets/cancel-icon.svg') + ')' }"></div>
              <div class="delete-icon" @click="showDeleteConfirmation(index, $event)" :style="{ backgroundImage: 'url(' + require('@/assets/delete-icon.svg') + ')' }"></div>
            </div>
          </div>
        </div>
      </div>
      <div class="pagination">
        <a href="#" class="page-link" @click.prevent="changePage(-1)">《</a>
        <a href="#" v-for="page in totalPages" :key="page" class="page-button" :class="{ active: currentPage === page }" @click.prevent="changePageTo(page)">{{ page }}</a>
        <a href="#" class="page-link" @click.prevent="changePage(1)">》</a>
      </div>
      <div class="frame-15">
        <div class="div-2">
          <div class="frame-16" @click="openAddModal"><div class="text-wrapper-9">+</div></div>
        </div>
        <div class="div-2">
          <div class="frame-17" @click="reloadPage"><div class="text-wrapper-10">reload</div></div>
        </div>
      </div>

      <!-- delete彈出視窗 -->
      <div v-if="showConfirmModal" :style="confirmModalStyle" class="delete-confirm-popup">
        <div class="arrow-up"></div> 
        <div class="exclamation-wrapper">
          <div class="exclamation-circle"><span class="exclamation">!</span></div> 
          <span class="popup-text">Sure to Delete?</span>
        </div>
        <div class="button-wrapper">
          <button class="cancel-button" @click="cancelDelete">Cancel</button>
          <button class="ok-button" @click="confirmDelete">OK</button>
        </div>
      </div>

      <!-- add彈出視窗 -->
      <div v-if="showAddModal" class="add-modal">
        <div class="modal-content">
          <span class="close" @click="closeAddModal">&times;</span>
          <div>
            <label for="newName">name:</label>
            <input type="text" id="newName" v-model="newItem.name">
          </div>
          <div>
            <label for="newValue">value:</label>
            <input type="text" id="newValue" v-model="newItem.value">
          </div>
          <div>
            <label for="newDescription">description:</label>
            <input type="text" id="newDescription" v-model="newItem.description">
          </div>
          <button class="send-button" @click="addNewItem">Send</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dataList: [
        { name: 'api_host_ip', value: '172.20.0.4', description: '172.20.0.4', editMode: false },
        { name: 'api_host_port', value: '8080', description: '8080', editMode: false },
        { name: 'api_host_uid', value: 'root', description: 'root', editMode: false },
        { name: 'api_host_pwd', value: 'test', description: 'test', editMode: false },
        { name: 'ackPacsType', value: '1', description: '1', editMode: false },
        { name: 'csh_medAnal_url', value: 'http://10.191.20.50:8085/AIresult/', description: 'http://10.191.20.50:8085/AIresult/', editMode: false },
        { name: 'db_host_ip', value: '172.20.0.5', description: '172.20.0.5', editMode: false },
        { name: 'db_port', value: '3306', description: '3306', editMode: false },
        { name: 'db_uid', value: 'root', description: 'root', editMode: false },
        { name: 'db_pwd', value: 'test', description: 'test', editMode: false },
        { name: 'db_dbname', value: 'v5_med_m1', description: 'v5_med_m1', editMode: false },
        { name: 'DiskRecycle_Type', value: '2', description: '2', editMode: false },
        { name: 'DiskRecycle_Limit', value: '360', description: '360', editMode: false },
        { name: 'isAutoFlow', value: false, description: 'false', editMode: false },
        { name: 'study_list_mode', value: 'product', description: 'product', editMode: false },
        { name: 'viewer_mode', value: 'product', description: 'product', editMode: false },
        { name: 'hospital', value: 'product', description: 'product', editMode: false },
        { name: 'hospital_group_name', value: 'product', description: 'product', editMode: false },
        { name: 'license_times', value: '5002', description: '5002', editMode: false },
        { name: 'mq_host', value: '172.20.0.6', description: '172.20.0.6', editMode: false },
        { name: 'mq_port', value: '5672', description: '5672', editMode: false },
        { name: 'mq_uid', value: 'v5super', description: 'v5super', editMode: false },
        { name: 'mq_pwd', value: 'v5m1', description: 'v5m1', editMode: false },
        { name: 'NoduleComparison', value: false, description: 'false', editMode: false },
        { name: 'dicom_data', value: '/data_volumns/dicom', description: '/data_volumns/dicom', editMode: false },
        { name: 'orthanc_instances', value: 'http://172.20.0.2:8042/instances', description: 'http://172.20.0.2:8042/instances', editMode: false },
        { name: 'orthanc_dicom_web_api', value: 'http://172.20.0.2:8042', description: 'http://172.20.0.2:8042', editMode: false },
        { name: 'DrawObject', value: '2', description: '2', editMode: false },
        { name: 'CoordinateSkew', value: '0', description: '0', editMode: false },
        { name: 'DoTIFF', value: '9', description: '9', editMode: false },
        { name: 'SaveHardDriveSpace', value: false, description: 'false', editMode: false },
        { name: 'dicomContentType', value: '7', description: '7', editMode: false },
        { name: 'storeDICOM', value: true, description: 'true', editMode: false },
        { name: 'OHIF_url', value: 'https://v5lungct2.v5.com.tw/viewer', description: 'https://v5lungct2.v5.com.tw/viewer', editMode: false },
        { name: 'postToPacsCrontab', value: false, description: 'false', editMode: false },
        { name: 'SpecificCharacterSet', value: 'ISO_IR 192', description: 'ISO_IR 192', editMode: false },
        { name: 'FromCharacterSet', value: 'BIG5', description: 'BIG5', editMode: false },
        { name: 'SC_Font', value: '3', description: '3', editMode: false },
        { name: 'CopyAccNumber', value: true, description: 'true', editMode: false },
        { name: 'redis_ip', value: '172.20.0.22', description: '172.20.0.22', editMode: false },
        { name: 'report_queue_name', value: 'wp_MED01-Report', description: 'wp_MED01-Report', editMode: false },
        { name: 'reDrawGsps', value: false, description: 'false', editMode: false },
        { name: 'vghtc_api', value: 'http://127.0.0.1:8080', description: 'http://127.0.0.1:8080', editMode: false },
        { name: 'SC_P5581', value: '0', description: '0', editMode: false },
        { name: 'loadunload', value: false, description: 'false', editMode: false },
        { name: 'lung_mask_data', value: '/etc/log/MED01-CAD', description: '/etc/log/MED01-CAD', editMode: false },
        { name: 'VOI_WindowCenter', value: '-600', description: '-600', editMode: false },
        { name: 'VOI_WindowWidth', value: '1600', description: '1600', editMode: false }
      ],
      showConfirmModal: false, 
      confirmModalStyle: {},   
      itemToDelete: null,     
      currentPage: 1,
      itemsPerPage: 9,
      showAddModal: false,
      newItem: { name: '', value: '', description: '' }, 
    };
  },
  computed: {
    paginatedData() {
      const startIndex = (this.currentPage - 1) * this.itemsPerPage;
      return this.dataList.slice(startIndex, startIndex + this.itemsPerPage);
    },
    totalPages() {
      return Math.ceil(this.dataList.length / this.itemsPerPage);
    },
  },
  methods: {
    changePage(delta) {
      const newPage = this.currentPage + delta;
      this.showConfirmModal = false; /* 翻頁時關閉 */
    },
    changePageTo(page) {
      this.currentPage = page;
      this.showConfirmModal = false;
    },


    toggleEditMode(index) {
      const realIndex = (this.currentPage - 1) * this.itemsPerPage + index;
      this.dataList[realIndex].editMode = !this.dataList[realIndex].editMode;
    },
    saveChanges(index) {
      const realIndex = (this.currentPage - 1) * this.itemsPerPage + index;
      this.dataList[realIndex].editMode = false;
    
    },
    cancelChanges(index) {
      const realIndex = (this.currentPage - 1) * this.itemsPerPage + index;
      this.dataList[realIndex].editMode = false;
   
    },
    toggleBooleanValue(index) {
      const realIndex = (this.currentPage - 1) * this.itemsPerPage + index;
      this.dataList[realIndex].value = !this.dataList[realIndex].value;
     
    },
    showDeleteConfirmation(index, event) {
    
      const { top, left, height } = event.target.getBoundingClientRect();

      
      const scrollTop = window.scrollY || window.pageYOffset; /* 滑scrollbar不會亂跑 */
      const scrollLeft = window.scrollX || window.pageXOffset;

     
      this.confirmModalStyle = {
        position: 'absolute',
        top: `${top + scrollTop - height - 70}px`,
        left: `${left + scrollLeft + 14}px`,
        transform: 'translateX(-50%)', 
      };

      this.showConfirmModal = true; 
      this.itemToDelete = (this.currentPage - 1) * this.itemsPerPage + index; 
    },
    confirmDelete() {
      this.dataList.splice(this.itemToDelete, 1); 
      this.showConfirmModal = false;               
    },
    cancelDelete() {
      this.showConfirmModal = false; 
    },
    openAddModal() {
      this.showAddModal = true; 
    },
    closeAddModal() {
      this.showAddModal = false; 
    },
    addNewItem() {
      if (this.newItem.name && this.newItem.value) {
        this.dataList.push({
          name: this.newItem.name,
          value: this.newItem.value,
          description: this.newItem.description,
          editMode: false,
        });
        this.newItem = { name: '', value: '', description: '' }; 
        this.showAddModal = false; 
      }
    },
    reloadPage() {
      location.reload(); 
    }
  }
};
</script>

<style>


.system-variable {
  background-color: var(--variable-collection-black);
  display: flex;
  flex-direction: column; 
  align-items: center; 
  width: 100%;
}

.system-variable .div {
  background-color: var(--variable-collection-black);
  width: 100%;
  height: 1080px;
  position: relative;
}

.system-variable .overlap-group {
  display: flex;
  align-items: center; 
  justify-content: space-between; 
  width: 100%; 
  top: 0;
  left: 0;
  padding: 20px 40px; 
  background-color: var(--variable-collection-white);
  z-index: 10; 
}

.system-variable .v {
  height: 58px;
  object-fit: cover;
}

.system-variable .frame {
  display: flex;
  flex-grow: 1;
  justify-content: center;
  align-items: center;
  gap: 30px;
}

.system-variable .div-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 0px 20px;
}

.system-variable .text-wrapper {
  font-family: "Inter-Medium", Helvetica;
  font-weight: 500;
  color: var(--variable-collection-black);
  font-size: 24px;
}

.system-variable .frame-2 {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0px 40px;
  background-color: var(--variable-collection-button);
}

.system-variable .text-wrapper-2 {
  font-family: "Inter-ExtraBold", Helvetica;
  font-weight: 800;
  color: var(--variable-collection-white);
  font-size: 24px;
  letter-spacing: 4.8px;
}

.system-variable .icon {
  position: absolute; 
  right: 40px; 
  height: 60px;
  width: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  border: 6px solid var(--variable-collection-v5-blue);
}

.system-variable .union {
  width: 80%;
  height: 80%;
  object-fit: contain;
}

.system-variable .frame {
  display: flex;
  width: 1012px;
  height: 73px;
  align-items: center;
  gap: 100px;
  position: absolute;
  top: 20px;
  left: 550px; 
}

.system-variable .img {
  flex: 0 0 auto;
}

.system-variable .frame-4 {
  display: flex;
  flex-direction: column;
  width: 2000px;
  align-items: flex-start;
  gap: 20px;
  position: absolute;
  top: 330px;
  left: 80px;
  z-index: 1; /* 層級低於 overlap-group */
  color: #ffffff; 
}

.system-variable .frame-5 {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0px 20px;
  position: relative;
  align-self: stretch;
  width: 100%;
  flex: 0 0 auto;
}

.system-variable .text-wrapper-3 {
  font-family: "Inter-Medium", Helvetica;
  font-weight: 100%;
  color: var(--variable-collection-white);
  font-size: 24px;
}

.system-variable .frame-6 {
  display: flex;
  min-width: 800px;
  align-items: center;
  position: relative;
  flex: 1;
}

.system-variable .text-wrapper-4 {
  width: 90px;
  font-family: "Inter-Medium", Helvetica;
  font-weight: 500;
  color: var(--variable-collection-white);
  font-size: 24px;
}

.system-variable .frame-7 {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 1px;
  align-self: stretch;
  width: 100%;
}
.system-variable .frame-8-title {
  display: flex;
  align-items: center;

  padding: 0px 20px;
  align-self: stretch;
  width: 100%;
  background-color: transparent;
  margin-top: 120px; 
}

.system-variable .text-wrapper-title {
  font-size: 24px; 
  font-family: "Inter-Bold", Helvetica;
  font-weight: 700;
  color: var(--variable-collection-white);
}

.system-variable .text-wrapper-title:nth-child(1) {
  margin-left: 78px; 
}

.system-variable .text-wrapper-title:nth-child(2) {
  margin-left: 523px; 
}

.system-variable .text-wrapper-title:nth-child(3) {
  margin-left: 548px; 
}

.system-variable .text-wrapper-title:nth-child(4) {
  margin-left: 608px; 
}

.system-variable .frame-8 {
  display: flex;
  height: 50px;
  align-items: center;
  padding: 0px 20px;
  align-self: stretch;
  width: 100%;
  background-color: var(--variable-collection-list-bg);
  border-bottom: 1px solid var(--variable-collection-purple-blue);
}

.system-variable .text-wrapper-5 {
  flex: 1;
  font-size: 20px;
  font-family: "Inter-Medium", Helvetica;
  font-weight: 500;
  color: var(--variable-collection-white);
}

.system-variable .value {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 10px;
  padding: 0px 20px 0px 0px;
  flex: 1;
}

.system-variable .frame-9 {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 20px 10px 0px;
  align-self: stretch;
  width: 100%;
}

.system-variable .text-wrapper-6 {
  width: fit-content;
  font-family: "Inter-Medium", Helvetica;
  font-weight: 500;
  color: var(--variable-collection-white);
  font-size: 20px;
}

.system-variable .div-2 {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 0 0 auto;
}

.system-variable .edit-icon, .delete-icon, .save-icon, .cancel-icon {
  width: 30px;
  height: 30px;
  background-size: 100% 100%;
  cursor: pointer;
  margin-right: 24px;
}
.save-icon, .cancel-icon {
  order: 1; 
}

.delete-icon {
  order: 2;
}

.system-variable .frame-15 {
  display: flex;
  width: 2100px;
  align-items: center;
  justify-content: space-between;
  position: absolute;
  top: 149px;
  left: 80px;
}

.system-variable .frame-16 {
  display: inline-flex;
  align-items: flex-start;
  justify-content: center;
  gap: 10px;
  padding: 8px 25px;
  flex: 0 0 auto;
  background-color: var(--variable-collection-button);
  border-radius: 6px;
  cursor: pointer; 
}

.system-variable .text-wrapper-9 {
  width: fit-content;
  font-family: "Inter-ExtraBold", Helvetica;
  font-weight: 800;
  color: var(--variable-collection-white);
  font-size: 32px;
}

.system-variable .frame-17 {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 15px 23px;
  flex: 0 0 auto;
  background-color: var(--variable-collection-button);
  border-radius: 6px;
  cursor: pointer; 
}

.system-variable .text-wrapper-10 {
  width: fit-content;
  font-family: "Inter-Medium", Helvetica;
  font-weight: 500;
  color: var(--variable-collection-white);
  font-size: 23px;
}

.edit-input {
  width: 400px; 
  box-sizing: border-box;
  padding: 5px; 
  border: 1px solid #ccc;
  border-radius: 4px; 
  outline: none; 
  font-family: "Inter-Medium", Helvetica;
  font-size: 20px;
  color: #ffffff; 
  background-color: #000000; 
  box-shadow: inset 0px 0px 5px rgba(0, 0, 0, 0.2); 
}

.edit-input:focus {
  border-color: #96c8ea; 
  box-shadow: 0px 0px 5px rgba(0, 116, 189, 0.5); 
}

.hidden {
  display: none;
}

.pagination {
  display: flex;
  justify-content: flex-end; 
  margin-top: 680px;
  padding: 20px 0;
  padding-right: 50px; 
}

.pagination button, .pagination a {
  margin: 0 5px;
  padding: 13px 18px;
  background-color: #ffffff;
  color: rgb(0, 0, 0);
  border: none;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
  display: inline-block;
}

.pagination button:disabled, .pagination a.disabled {
  background-color: #ddd;
  cursor: not-allowed;
}

.pagination .page-button {
  margin: 0 7px;
  padding: 11px 18px;
  background-color: #fff;
  color: black;
  border: 1px solid #ddd; 
  border-radius: 5px;
  cursor: pointer;
  font-size: 18px; 
}

.pagination .page-button.active {
  background-color: #0892D0;
  color: white;
  border: 1px solid #0892D0; 
  font-size: 18px; 
}

/* delete視窗 */
.arrow-up {
  width: 0; 
  height: 0; 
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  border-top: 10px solid #ffffff;
  position: absolute;
  bottom: -10px; 
  left: 50%;
  transform: translateX(-50%);
  z-index: 1001;
}

.delete-confirm-popup {
  background-color: #fff;
  border-radius: 4px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  padding: 10px;
  position: absolute;
  width: 180px;
  z-index: 1000;
  text-align: center;
}

.delete-confirm-popup .exclamation-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
}

.delete-confirm-popup .exclamation-circle {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #ffa800;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  margin-right: 5px;
}

.delete-confirm-popup .exclamation {
  color: #fff;
  font-weight: bold;
  font-size: 14px;
}

.delete-confirm-popup .popup-text {
  margin-top: 5px;
  margin-bottom: 10px;
  font-size: 14px;
  color: #000;
}

.delete-confirm-popup .cancel-button,
.delete-confirm-popup .ok-button {
  background-color: #0892d0;
  border: none;
  border-radius: 4px;
  color: #fff;
  cursor: pointer;
  padding: 5px 10px;
  margin: 5px;
  font-size: 14px;
}

.delete-confirm-popup .cancel-button {
  background-color: #ccc;
  color: #000;
}

/* add視窗 */
.add-modal {
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  z-index: 1050;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); 
}

.modal-content {
  background-color: #6c6969;
  padding: 40px 30px;
  border-radius: 4px;
  width: 570px;
  border-radius: 12px;
  text-align: center;
  color: white;
  position: relative;
}

.modal-content label {
  display: block;
  font-size: 25px; 
  margin-bottom: 10px;
  
}

.modal-content input {
  width: calc(100% - 40px);
  padding: 15px 10px;
  margin-bottom: 20px;
  border-radius: 4px;
  font-size: 20px;
  border: 1px solid #ccc;
}

.send-button {
  background-color: #0892D0;
  border: none;
  padding: 15px 10px;
  border-radius: 4px;
  width: 550px;
  font-size: 22px; 
  color: #fff;
  cursor: pointer;
}

.close {
  position: absolute;
  top: 10px;
  right: 20px;
  font-size: 33px; 
  color: white;
  cursor: pointer;
}

/* tf 按鈕 */
.toggle-container {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.frameParent {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  border: 1px solid #ffffff;
  border-radius: 8px;
  box-sizing: border-box;
  overflow: hidden;
}

.trueWrapper  {
align-self: stretch;
width: 104px;
border-radius: 8px;
background-color: #34b539;
border: 2px #fff;
box-sizing: border-box;
display: flex;
flex-direction: row;
align-items: center;
justify-content: center;
padding: 10px 20px;
}

.falseWrapper 
  {
  align-self: stretch;
  width: 113px;
  border-radius: 8px;
  background-color: #e45f5f;
  border: 2px solid #fff;
  box-sizing: border-box;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  padding: 10px 20px;
}


.trueWrapper.active {
  opacity: 1;
}

.falseWrapper.active {
  opacity: 1;
}

.trueWrapper:not(.active),
.falseWrapper:not(.active) {
  opacity: 0.5;
}

</style>
 