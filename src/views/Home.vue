<template>
  <div id="app">
      <Menu />

        <img alt="Vue logo" src="../assets/logo.png">
        <h1><b>Welcome {{nameField}}</b></h1>
        <a-spin v-if="loading" size="large"></a-spin>
        <a-timeline mode="alternate">
          <div v-if="completeRoster === true">
            <a-timeline-item color="green">Complete Roster</a-timeline-item>
          </div>
          <div v-else>
            <a-timeline-item color="red">Complete Roster</a-timeline-item>
          </div>
          <div v-if="adminApproval === true">
            <a-timeline-item color="green">Admin Approval</a-timeline-item>
          </div>
          <div v-else>
            <a-timeline-item color="red">Admin Approval</a-timeline-item>
          </div>
             <div v-if="paymentSuccess === true">
            <a-timeline-item color="green">Payment</a-timeline-item>
          </div>
          <div v-else>
            <a-timeline-item color="red">Payment</a-timeline-item>
          </div>

          <div v-if="completeRoster === true && adminApproval === true && paymentSuccess === false">
            <a-timeline-item color="#00CCFF">
              <template #dot>
                <DollarCircleOutlined style="font-size: 16px" />
              </template>
              <p>Make a payment $$$</p>
            </a-timeline-item>
          </div>
          <div v-else-if="completeRoster === true && adminApproval === true && paymentSuccess === true">
            <a-timeline-item color="#00CCFF">
              <template #dot>
                <SmileOutlined />
              </template>
              <p>Get Ready to Play!</p>
            </a-timeline-item>
          </div>
          <div v-else-if="completeRoster === true && adminApproval === false && paymentSuccess === false">
            <a-timeline-item color="#00CCFF">
              <template #dot>
                <ClockCircleOutlined />
              </template>
              <p>Waiting for admin approval!</p>
            </a-timeline-item>
          </div>
          <div v-else>
            <a-timeline-item color="#00CCFF">
              <template #dot>
                <FormOutlined />
              </template>
              <p>Signup your team!</p>
            </a-timeline-item>
          </div>
        </a-timeline>

        <div v-if="completeRoster === true && adminApproval === true && paymentSuccess === false">
            <CashApp />
        </div>

            <a-modal
                v-model:visible="visible"
                title="Name, please?"
                width="75%"
                wrap-class-name="full-modal"
                @ok="handleSubmission"
              >
                <div v-html="modalText"></div>
                      <a-input v-model:value="nameField" />
              </a-modal>
  </div>
</template>


<script>
import { ref, onBeforeMount, toRaw } from 'vue';
import firebase from "firebase/app"
import 'firebase/auth';
import Menu from '../components/Menu.vue';
import CashApp from '../components/CashApp.vue';
import { message } from 'ant-design-vue';
import { SmileOutlined, ClockCircleOutlined, FormOutlined, DollarCircleOutlined } from '@ant-design/icons-vue';

export default {
  setup() {
      const completeRoster = ref(``);
      const adminApproval = ref(``);
      const paymentSuccess = ref(``);
      const modalText = ref('default');
      const visible = ref(false);
      const nameField = ref('');
      const loading = ref(true);
      const auth = firebase.auth();

    onBeforeMount(() => {
      getIdToken()
    })

    const getUserInfo = async (token) => {
      const response = await fetch(`https://us-central1-wffa25444.cloudfunctions.net/teamData?api=getUserData&type=user`, {
        method:'GET',
        headers:{
            Authorization:"Bearer "+token,
            "Content-Type": "application/json"
            }
      })
    
      if (response.status === 200) {
        return response.json()
        }
      else { 
            message.error({
                          content: `ERROR`,
                          duration: 2,
              }); 
          }
      }

    const getIdToken = async () => {
        auth.onAuthStateChanged(async (user) => {
          if (user) {
            const token = await user.getIdToken();
            const res = await getUserInfo(token);
            completeRoster.value = res.data[0].teamSignup
            adminApproval.value = res.data[0].approve
            paymentSuccess.value = res.data[0].payment
            if (res.data[0].name === null) {
              setUsersName()
            } else {
              nameField.value = res.data[0].name
              loading.value = false
            }
          }
      })
    }

    const handleSubmission = async () => {
      const user = auth.currentUser;
      const idToken = await user.getIdToken();
      const rosterPayload = {
          "data": toRaw(nameField.value)
      }
      const response = await (await fetch(`https://us-central1-wffa25444.cloudfunctions.net/teamData?api=updateUserProfile&type=addName`, {
        method:'POST',
        body: JSON.stringify(rosterPayload),
        headers:{
            Authorization:"Bearer "+idToken,
            "Content-Type": "application/json"
            }
        }))
        if (response.status === 200) {
            message.success({
              content: 'Thank you!',
              duration: 2,
            }); 
          }
        else { 
              message.error({
                        content: `ERROR`,
                        duration: 2,
            }); 
         }
        visible.value = false;
      }

    const setUsersName = () => {
      loading.value = false
      visible.value = true;
      modalText.value = `<div style="align: center"><h3>Enter your first name + last name:</h3> <br />`
    }

    return {
      completeRoster,
      adminApproval,
      paymentSuccess,
      visible,
      handleSubmission,
      modalText,
      nameField,
      loading
    };
  },

  components: {
   Menu,
   CashApp,
   SmileOutlined,
   ClockCircleOutlined,
   FormOutlined, 
   DollarCircleOutlined
  },
};
</script>
<style>


img {
    width: 250px;
    height: 200px;
    display: block;
    margin-left: auto;
    margin-right: auto;
}


</style>

