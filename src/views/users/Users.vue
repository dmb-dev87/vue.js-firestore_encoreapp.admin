<template>
  <CRow>
    <CCol col="12" xl="12">
      <CCard>
        <CCardHeader>
          <CRow>
            <CCol>
              Users
            </CCol>
            <CCol sm="3" class="text-right">
              <CButton class="px-3" color="success" @click="goAddUser">Add User</CButton>
            </CCol>
          </CRow>
        </CCardHeader>
        <CCardBody>
          <CDataTable
            hover
            striped
            :items="items"
            :fields="fields"
            :items-per-page="5"
            clickable-rows
            :active-page="activePage"
            :pagination="{ doubleArrows: false, align: 'center'}"
            @page-change="pageChange"
          >
            <template #profileImage="data">
              <td>
                <img :src="data.item.profileImage" height="50px" />
              </td>
            </template>
            <template #action="{item}">
              <td>
                <CLink block color="link" class="text-left" :to="{name: 'User', params: { id: item.key }}">Details</CLink>
              </td>
            </template>
          </CDataTable>
        </CCardBody>
      </CCard>
    </CCol>
  </CRow>
</template>

<script>
import { db, auth } from './../../firebase.js'
export default {
  name: 'Users',
  data () {
    return {
      items: [],
      fields: [
        { key: 'profileImage', label: 'Avatar'},
        { key: 'username', label: 'Name', _classes: 'font-weight-bold' },
        { key: 'country' },
        { key: 'email' },
        { key: 'gender' },
        { key: 'phone'},
        { key: 'action'}
      ],
      activePage: 1,
      currentUser: {}
    }
  },
  async created() {
    await this.getCurrentUser()
    let dbRef
    if (this.currentUser.userrole == "admin") {
      dbRef = db.collection('users')
    } else {
      dbRef = db.collection('users')
        .where('uid', '==', auth.currentUser.uid)
    }

    dbRef.get()
      .then(querySnapshot => {
        this.items = []
        querySnapshot.docs.map((doc) => {
          this.items.push({
            key: doc.id,
            username: `${doc.data().firstname ? doc.data().firstname : '' } ${doc.data().lastname ? doc.data().lastname : ''}`,
            country: doc.data().country ? doc.data().country : '',
            email: doc.data().email ? doc.data().email : '',
            gender: doc.data().gender ? doc.data().gender : '',
            phone: doc.data().mobilenumber ? doc.data().mobilenumber : '',
            profileImage: doc.data().profileImage,
          })
        })
    })
  },
  watch: {
    $route: {
      immediate: true,
      handler (route) {
        if (route.query && route.query.page) {
          this.activePage = Number(route.query.page)
        }
      }
    }
  },
  methods: {
    async getCurrentUser() {
      const query = db.collection('users').doc(auth.currentUser.uid)
      await query.get()
        .then((doc) => {
          this.currentUser = doc.data()
        })
    },
    goAddUser() {
      this.$router.push({path: '/user/add'})
    },
    getBadge (status) {
      switch (status) {
        case 'Active': return 'success'
        case 'Inactive': return 'secondary'
        case 'Pending': return 'warning'
        case 'Banned': return 'danger'
        default: 'primary'
      }
    },
    pageChange (val) {
      this.$router.push({ query: { page: val }})
    }
  }
}
</script>
