<template>
    <vuci-form :uci-config="config" v-if="showModal">
        <vuci-named-section :name="modalData" v-slot="{ s }">
            <vuci-form-item-select label="Protocol" name="proto" initial="static" :options="protocols" :uci-section="s"/>
                <vuci-form-item-input  label="IP Address" :uci-section="s" name="address" depend="proto == 'static'" rules="ipaddr" required/>
                <vuci-form-item-input label="Netmask" :uci-section="s" name="netmask" depend="proto == 'static'" rules="netmask4" required/>
                <vuci-form-item-input label="Gateway"  name="gateway" depend="proto == 'static'" :uci-section="s" rules="ipaddr"/>
                <vuci-form-item-list label="DNS" name="dns" depend="proto == 'static'" :uci-section="s"/>
                <vuci-form-item-switch label="DHCP" name="dhcp" depend="proto == 'static'" :initial="true" :uci-section="s"/>
        </vuci-named-section>
    </vuci-form>
    
</template>

<script>
export default {
    props: {
        modalData: String,
        showModal: Boolean
    },
     data () {
    return {
      config: 'example',
      section: 'interface',
      protocols: [
        ['dhcp', 'DHCP Client'],
        ['static', 'Static address']
      ],
    }
  },
}
</script>