<template>
  <div id="container">
      <div class="page-header">
         <h1 class="text-center">연락처 관리 애플리케이션</h1>
      </div>
      <component :is="currentView" :contact="contact"></component>
      <contactList :contactlist="contactlist"></contactList>
  </div>
</template>

<script>
import ContactList from './components/ContactList';
import AddContact from './components/AddContact';
import UpdateContact from './components/UpdateContact';
import UpdatePhoto from './components/UpdatePhoto';
import CONF from './Config.js';

export default {    
    name: 'app',
    components : { ContactList, AddContact, UpdateContact, UpdatePhoto },
    data: function() {
        return { 
            currentView : null, 
            contact : { no:0, name:'', tel:'', address:'', photo:'' },
            contactlist : {
                pageno:1, pagesize: CONF.PAGESIZE, totalcount:0, contacts:[]
            }
        }
    },
    methods : {
      pageChanged : function(page) {
          this.contactlist.pageno = page;
          this.fetchContacts();
      },
      fetchContacts : function() {
          this.$axios.get(CONF.FETCH, {
              params : { 
                  pageno:this.contactlist.pageno, 
                  pagesize:this.contactlist.pagesize 
              } 
          })
          .then((response) => {
              this.contactlist = response.data;
          })
          .catch(()=> {
              this.contactlist.contacts = [];
          })
      },
      addContact : function(contact) {
          this.$axios.post(CONF.ADD,  contact)
          .then((response) => {
            if (response.data.status === "success") {
              this.contactlist.pageno = 1;
              this.fetchContacts();
            } 
          })
          .catch(()=> {
          })
      },
      updateContact : function(contact) {
          this.$axios.put(CONF.UPDATE.replace("${no}", contact.no), contact)
          .then((response) => {
            if (response.data.status === "success") {
              this.fetchContacts();
            } 
          })
          .catch(() => {
          })
      },
      fetchContactOne : function(no) {
          this.$axios.get(CONF.FETCH_ONE.replace("${no}", no))
          .then((response) => {
              this.contact = response.data;
          })
          .catch(()=> {
          })
      },
      deleteContact : function(no) {
          this.$axios.delete(CONF.DELETE.replace("${no}", no))
          .then((response) => {
            if (response.data.status === "success") {
              this.fetchContacts();
            } 
          })
          .catch(() => {
          })
      },
      updatePhoto : function(no, file) {
          var data = new FormData();
          data.append('photo', file);
          this.$axios.post(CONF.UPDATE_PHOTO.replace("${no}", no), data)
          .then((response) => {
            if (response.data.status === "success") {
              this.fetchContacts();
            } 
          })
          .catch(() => {
          });
      }
    }
}
</script>

<style scoped>
#container {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>