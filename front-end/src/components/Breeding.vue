<template lang="html">
<div class="appContent">
  <div  v-cloak class="card shadow">
    <div class="card-header text-white bg-primary">
      <div class="row no-gutters">
        <div class="col-8">
          <h3 class="font-weight-bold">{{msg}}</h3>
        </div>
        <div class="col-4">
          <h5 class="text-right"><a href="#"  class="text-white" data-toggle="modal" data-target="#addBreedingModal">New Breeding Record</a></h5>
        </div>
      </div>
    </div>
    <template v-if="breedings.length > 0">
      <div class="table-responsive">
        <table class="table table-striped table-hover">
          <thead class="thead-custom-darkgray">
            <tr>
              <th>Ear Tag</th>
              <th>Date</th>
              <th>Method</th>
              <th>Sire</th>
              <th>Edit</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="breeding in breedings" >
              <td>{{breeding.tag_id}}</td>
              <td>{{breeding.date}}</td>
              <td>{{breeding.method}}</td>
              <td>{{breeding.sire}}</td>
              <td><router-link :to="{name: 'breeding-event', params:{user, id: breeding._id}}"><i class="fa fa-2x fa-chevron-circle-right"></i></router-link></td>
            </tr>
          </tbody>
        </table>
      </div>
    </template>

    <template v-else>
      <h6 class= "p-3">There is no breeding data to show. Get started by clicking "New Breeding Record" above.</h6>
    </template>

  </div>

  <!-- ADD Modal -->
    <div class="modal fade" id="addBreedingModal" tabindex="-1" role="dialog" aria-labelledby="addBreedingModalLabel" aria-hidden="true">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header card-header">
            <h5 class="modal-title text-primary font-weight-bold" id="addBreedingModalLabel">Add new breeding record</h5>
            <button type="button" @click="clear()" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
        <form>
          <div class="modal-body">
            <div class="container-fluid">
              <div class="row">
                <div class="col-md-4 col-sm-0">
                </div>
                <div class="col-md-8 col-sm-12 ">
                  <small class="float-right text-secondary mb-3">Fields marked with (*) are required.</small>
                </div>
              </div>
            </div>
            <div class="errorContainer text-danger">
              <p v-if="errors.length">
                <b>Please correct the following error(s):</b>
                <ul>
                  <li v-for="error in errors">{{ error }}</li>
                </ul>
              </p>
            </div>
            <div class="form-group">
              <label for="addBreedingTagId">Ear Tag Number*</label>
              <select v-model="newBreeding.tag_id"  class="form-control" id="addBreedingTagId">
                <option disabled value="">Select an active animal</option>
                <option v-for="cow in cows" v-if="cow.status =='Active' && cow.type =='Cow'">{{cow.tag_id}}</option>
              </select>
            </div>
            <div class="form-group">
              <label for="addBreedingDate">Breeding date*</label>
              <input v-model="newBreeding.date" type="date" class="form-control" id="addBreedingDate" placeholder="mm/dd/yyyy">
            </div>
            <div class="form-group">
              <label for="addBreedingMethod">Method*</label>
              <select v-model="newBreeding.method" class="form-control" id="addBreedingMethod">
                <option disabled value="">Select breeding method</option>
                <option>AI</option>
                <option>Embryo</option>
                <option>Natural</option>
              </select>
            </div>
            <div class="form-group">
              <label for="addBreedingSire">Sire*</label>
              <input v-model="newBreeding.sire" type="text" class="form-control" id="addBreedingSire" placeholder="Sire">
            </div>
            <div class="form-group">
              <label for="addBreedingTechnician">Technician</label>
              <input v-model="newBreeding.technician" type="text" class="form-control" id="addBreedingTechnician" placeholder="Technician">
            </div>
            <div class="form-group">
              <label for="addBreedingComments">Comments</label>
              <input v-model="newBreeding.comments" type="text" class="form-control" id="addBreedingComments" placeholder="Comments">
            </div>
            <div class="errorContainer text-danger">
              <p v-if="errors.length">
                <b>Please correct the following error(s):</b>
                <ul>
                  <li v-for="error in errors">{{ error }}</li>
                </ul>
              </p>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal" @click="clear()">Cancel</button>
            <button type="button" class="btn btn-primary addBtn" @click="checkForm($event); addBreeding();">Add to database</button>
          </div>
        </form>
      </div>
    </div>
  </div>
<!-- ADD Modal -->
</div>
</template>

<script>
import axios from 'axios';
import firebase from 'firebase';
import { clearModal } from './mixins/clearModal';
import { hideModal } from './mixins/hideModal';
import { authorization } from './mixins/auth';

export default {
  data() {
    return {
      msg: 'Herd Breeding',
      breedings: [],
      cows: [],
      user: null,
      errors: [],
      newBreeding: {
        tag_id: "",
        date: "",
        method: "",
        sire: "",
        technician: "",
        comments: ""
      }
    }
  },
  mixins: [hideModal, clearModal, authorization],
  created() {
    this.user = firebase.auth().currentUser.uid;
    this.fetchData();
  },
  methods: {
    addBreeding() {
      let newBreeding = {
        tag_id: this.newBreeding.tag_id,
        user: this.user,
        date: this.newBreeding.date,
        method: this.newBreeding.method,
        sire: this.newBreeding.sire,
        technician: this.newBreeding.technician,
        comments: this.newBreeding.comments
      }
      console.log(newBreeding);
      axios.post('http://127.0.0.1:3000/' + this.$route.params.user + '/breeding', newBreeding)
        .then((response) => {
          console.log(response);
          this.hideModal();
          this.clearModal();
          this.newBreeding.tag_id = "";
          this.newBreeding.date = "";
          this.newBreeding.method = "";
          this.newBreeding.sire = "";
          this.newBreeding.technician = "";
          this.newBreeding.comments = "";
          this.errors = [];
          this.fetchData();
        })
        .catch((err) => {
          console.log(err);
        });
    },
    checkForm: function(e) {
      if (this.newBreeding.tag_id && this.newBreeding.date && this.newBreeding.method && this.newBreeding.sire) return true;
      this.errors = [];
      if (!this.newBreeding.tag_id) this.errors.push("Ear tag is required.");
      if (!this.newBreeding.date) this.errors.push("Breeding date is required.");
      if (!this.newBreeding.method) this.errors.push("Breeding method is required.");
      if (!this.newBreeding.sire) this.errors.push("Sire is required.");
      e.preventDefault();
    },
    clear() {
      this.clearModal();
      this.newBreeding.tag_id = "";
      this.newBreeding.date = "";
      this.newBreeding.method = "";
      this.newBreeding.sire = "";
      this.newBreeding.technician = "";
      this.newBreeding.comments = "";
      this.errors = [];
    },
    fetchData() {
      axios.get('http://127.0.0.1:3000/' + this.$route.params.user + '/breeding')
        .then((response) => {
          this.breedings = response.data
        });
      axios.get('http://127.0.0.1:3000/' + this.$route.params.user + '/cattle')
        .then((response) => {
          this.cows = response.data
        });
    }
  },
  watch: {
    '$route': 'fetchData'
  }
}
</script>

<style lang="css">


</style>
