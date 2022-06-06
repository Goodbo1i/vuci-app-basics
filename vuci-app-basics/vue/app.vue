<template>
  <div >
    <vuci-form :uci-config="config" :key="rerender">
      <vuci-typed-section :type="section" :columns="columns" >
        <template #interface_name="{ s }" >
          <vuci-form-item-dummy :uci-section="s"  name="interface_name" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy :uci-section="s"  name="address" rules="ipaddr"/>
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy :uci-section="s"  name="netmask" rules="netmask4"/>
        </template>
        <template #action="{ s }">
          <a-button type="primary" size="small" :uci-section="s" style="margin-right: 5px" @click="editInterface(s['.name'])"> Edit </a-button>
          <a-popconfirm @confirm="del(s['.name'])" placement="left" :title="'Delete this interface?'">
            <a-button type="danger" size="small" :uci-section="s">Delete</a-button>
          </a-popconfirm>
        </template>
      </vuci-typed-section>
    </vuci-form>
    <template>
        <a-input v-model="interfaceName" placeholder="Interface Name" />
        <a-button type="primary" size="small" style="margin-top: 10px" @click="handleAdd">+ Add interface</a-button>
    </template>
    <a-modal  v-model="editModal" :width="800" >
      <sectionModal :modalData="sectionToEdit" :showModal="editModal"/>
        <template #footer></template>
    </a-modal>
  </div>
</template>

<script>
import sectionModal from './interfaces/sectionModal.vue'
export default {
  components: {
    sectionModal
  },
  data () {
    return {
      config: 'example',
      section: 'interface',
      sections: [],
      interfaceName: '',
      columns: [
        { name: 'interface_name', label: 'Interface name' },
        { name: 'address', label: 'IP address changed' },
        { name: 'netmask', label: 'Netmask changed' },
        { name: 'action', label: '#' }
      ],
      editModal: false,
      sectionToEdit: undefined,
      rerender: 1
    }
  },
  methods: {
    editInterface (sectionName) {
      this.sectionToEdit = sectionName
      this.editModal = true
    },
    del (name) {
      this.$spin()
      this.$uci.del(this.config, name)
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.load()
          this.$spin(false)
        })
      })
    },
    add (name) {
      this.$spin()
      const sid = this.$uci.add(this.config, this.section)
      this.$uci.set(this.config, sid, 'interface_name', name)
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.load()
          this.$spin(false)
        })
      })
    },
    handleAdd () {
      if (!this.interfaceName) return
      if (this.interfaceName.match(/^[a-zA-Z0-9_]+$/) === null) {
        this.$message.error(this.$t('validator.uci'))
        return
      }
      for (let i = 0; i < this.sections.length; i++) {
        if (this.sections[i].name === this.interfaceName) {
          this.$message.error(this.$t('Name already used'))
          return
        }
      }
      this.add(this.interfaceName)
    },
    async load () {
      await this.$uci.load(this.config)
      this.sections = this.$uci.sections(this.config, this.section)
      this.rerender++
    }
  },
  watch: {
    editModal () {
      this.load()
    }
  }
}
</script>
