<template>
  <div>
    <header class="page-header">
      <h1 class="page-title">License Check - Licenses</h1>
      <div class="page-description">Add and administer licenses, allow or disallow globally.</div>
      <div class="page-actions">
        <div class="button-group">
          <button id="item-add" @click="showAddDialog()">Add License</button>
        </div>
      </div>
    </header>
    <div>
      <div class="panel panel-vertical bordered-bottom spacer-bottom">
        <button class="search-box-submit button-clean"><i class="icon-search"></i></button>
        <input v-model="searchText" class="search-box-input" type="search" maxlength="100" placeholder="Search" autocomplete="off">
      </div>
    </div>
    <div>
      <table class="data zebra" width="100%">
        <thead>
          <tr>
            <th>Identifier</th>
            <th>Name</th>
            <th>Allowed</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in displayedItems" :key="item.identifier">
            <td class="thin">{{item.identifier}}</td>
            <td>{{item.name}}</td>
            <td class="thin">{{item.status}}</td>
            <td class="thin">
              <a class="icon-edit" @click="showEditDialog(item)" title="Edit item"></a>
              <a class="icon-delete" @click="showDeleteDialog(item)" title="Delete item"></a>
            </td>
          </tr>
          <tr v-show="!displayedItems">
            <td colspan="4">no items found</td>
          </tr>
        </tbody>
      </table>
    </div>
    <modal-dialog :header="editMode === 'add' ? 'Add License' : 'Edit License'" :show="!!itemToEdit" @close="cancelEdit()">
      <div slot="body" v-if="itemToEdit">
        <div class="modal-field">
          <label for="itemIdentifierEdit">Identifier<em class="mandatory">*</em></label>
          <input required v-focus="editMode === 'add'" :disabled="editMode !== 'add'" v-model="itemToEdit.identifier" id="itemIdentifierEdit" name="itemIdentifierEdit" type="text" size="50"
            maxlength="255">
        </div>
        <div class="modal-field">
          <label for="itemNameEdit">Name<em class="mandatory">*</em></label>
          <input required v-focus="editMode !== 'add'" v-model="itemToEdit.name" id="itemNameEdit" name="itemNameEdit" type="text" size="50" maxlength="255">
        </div>
        <div class="modal-field">
          <label for="itemStatusEdit">Status<em class="mandatory">*</em></label>
          <select required v-model="itemToEdit.status" id="itemStatusEdit" name="itemStatusEdit">
            <option value="true">true</option>
            <option value="false">false</option>
          </select>
        </div>
      </div>
      <span slot="footer"><button @click="saveItem(itemToEdit)">Save</button></span>
    </modal-dialog>
    <modal-dialog header="Delete license" :show="!!itemToDelete" @close="cancelDelete()">
      <div slot="body" v-if="itemToDelete">Are you sure you want to delete the license &quot;{{itemToDelete.identifier}}&quot; / &quot;{{itemToDelete.name}}&quot;?</div>
      <span slot="footer"><button @click="deleteItem(itemToDelete)">Delete</button></span>
    </modal-dialog>
  </div>
</template>

<script>
const focus = {
  inserted(el, binding) {
    if (binding.value)
      el.focus();
  },
};

export default {
  data() {
    return {
      items: [],
      itemToDelete: null,
      itemToEdit: null,
      editMode: null,
      searchText: null
    };
  },
  computed: {
    displayedItems() {
      if (!this.searchText || this.searchText.length == 0) {
        return this.items;
      }

      let search = this.searchText.toLowerCase();
      return this.items.filter(
        item =>
          item.name.toLowerCase().indexOf(search) >= 0 ||
          item.identifier.toLowerCase().indexOf(search) >= 0
      );
    }
  },
  created() {
    this.load();
  },
  methods: {
    load() {
      window.SonarRequest
        .getJSON("/api/licensecheck/licenses/show")
        .then(response => {
          this.items = response;
        });
    },
    showAddDialog() {
      this.itemToEdit = {};
      this.editMode = 'add';
    },
    showEditDialog(item) {
      this.itemToEdit = Object.assign({}, item);
      this.editMode = 'edit';
    },
    cancelEdit() {
      this.itemToEdit = null;
    },
    saveItem(item) {
      window.SonarRequest
        .post(`/api/licensecheck/licenses/${this.editMode}`, item)
        .then(() => {
          this.load()
        });
      this.itemToEdit = null;
    },
    showDeleteDialog(item) {
      this.itemToDelete = item;
    },
    cancelDelete() {
      this.itemToDelete = null;
    },
    deleteItem(item) {
      window.SonarRequest
        .post('/api/licensecheck/licenses/delete', { identifier: item.identifier })
        .then(() => {
          this.load()
        });
      this.itemToDelete = null;
    }
  },
  directives: { focus }
};
</script>
