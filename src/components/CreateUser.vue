<template>
  <div class="content">
    <form novalidate class="md-layout" @submit.prevent="validateUser(userId)">
      <md-card class="md-layout-item md-size-50 md-small-size-100">
        <md-card-header>
          <div class="md-title">Users</div>
        </md-card-header>
        <md-card-content>
          <div class="md-layout md-gutter">
            <div class="md-layout-item md-small-size-100">
              <md-field :class="getValidationClass('firstName')">
                <label for="first-name">First Name</label>
                <md-input name="first-name" id="first-name" autocomplete="given-name" v-model="form.firstName" :disabled="sending" />
                <span class="md-error" v-if="!$v.form.firstName.required">The first name is required</span>
                <span class="md-error" v-else-if="!$v.form.firstName.minlength">Invalid first name</span>
              </md-field>
            </div>

            <div class="md-layout-item md-small-size-100">
              <md-field :class="getValidationClass('lastName')">
                <label for="last-name">Last Name</label>
                <md-input name="last-name" id="last-name" autocomplete="family-name" v-model="form.lastName" :disabled="sending" />
                <span class="md-error" v-if="!$v.form.lastName.required">The last name is required</span>
                <span class="md-error" v-else-if="!$v.form.lastName.minlength">Invalid last name</span>
              </md-field>
            </div>
          </div>

          <div class="md-layout md-gutter">
            <div class="md-layout-item md-small-size-100">
              <md-field :class="getValidationClass('gender')">
                <label for="gender">Gender</label>
                <md-select name="gender" id="gender" v-model="form.gender" md-dense :disabled="sending">
                  <md-option></md-option>
                  <md-option value="M">Mail</md-option>
                  <md-option value="F">Famail</md-option>
                </md-select>
                <span class="md-error">The gender is required</span>
              </md-field>
            </div>

            <div class="md-layout-item md-small-size-100">
              <md-field :class="getValidationClass('age')">
                <label for="age">Age</label>
                <md-input type="number" id="age" name="age" autocomplete="age" v-model="form.age" :disabled="sending" />
                <span class="md-error" v-if="!$v.form.age.required">The age is required</span>
                <span class="md-error" v-else-if="!$v.form.age.maxlength">Invalid age</span>
              </md-field>
            </div>
          </div>

          <md-field :class="getValidationClass('email')">
            <label for="email">Email</label>
            <md-input type="email" name="email" id="email" autocomplete="email" v-model="form.email" :disabled="sending" />
            <span class="md-error" v-if="!$v.form.email.required">The email is required</span>
            <span class="md-error" v-else-if="!$v.form.email.email">Invalid email</span>
          </md-field>
        </md-card-content>

        <md-progress-bar md-mode="indeterminate" v-if="sending" />

        <md-card-actions>
          <md-button type="submit" class="md-primary" :disabled="sending">{{editUser ? "Edit" : "Add user"}}</md-button>
        </md-card-actions>
      </md-card>

      <md-snackbar :md-active.sync="userSaved">The user {{ lastUser }} was saved with success!</md-snackbar>
    </form>
    <div class="users">
      <md-table class="user">
        <md-table-row>
          <md-table-head>First Name</md-table-head>
          <md-table-head>Last Name</md-table-head>
          <md-table-head>Email</md-table-head>
          <md-table-head>Gender</md-table-head>
          <md-table-head>Job Title</md-table-head>
          <md-table-head>Actions</md-table-head>
        </md-table-row>
        <md-table-row v-for="(row, index) in rows">
          <md-table-cell>{{row.firstName}}</md-table-cell>
          <md-table-cell>{{row.lastName}}</md-table-cell>
          <md-table-cell>{{row.email}}</md-table-cell>
          <md-table-cell>{{row.gender == "F" ? "Famail" : "Mail" }}</md-table-cell>
          <md-table-cell>{{row.age}}</md-table-cell>
          <md-table-cell>
            <md-button class="md-icon-button md-accent" v-on:click="editRow(index)">
              <md-icon>edit</md-icon>
            </md-button>
            <md-button class="md-icon-button md-accent" v-on:click="deleteRow(index)">
              <md-icon>delete</md-icon>
            </md-button>
          </md-table-cell>
        </md-table-row>
      </md-table>
    </div>
  </div>
</template>

<script>
  import { validationMixin } from 'vuelidate'
  import {
    required,
    email,
    minLength,
    maxLength
  } from 'vuelidate/lib/validators'

  export default {
    name: 'FormValidation',
    mixins: [validationMixin],
    data: () => ({
      form: {
        firstName: null,
        lastName: null,
        gender: null,
        age: null,
        email: null,
      },
      userSaved: false,
      sending: false,
      lastUser: null,
      rows: [
        {firstName: "Vova", lastName: "Poghosyan", email: "vova@gmail.com", gender: "M", age: "22"}
      ],
      editUser: false,
      userId: null
    }),
    validations: {
      form: {
        firstName: {
          required,
          minLength: minLength(3)
        },
        lastName: {
          required,
          minLength: minLength(3)
        },
        age: {
          required,
          maxLength: maxLength(3)
        },
        gender: {
          required
        },
        email: {
          required,
          email
        }
      }
    },
    methods: {
      getValidationClass (fieldName) {
        const field = this.$v.form[fieldName]

        if (field) {
          return {
            'md-invalid': field.$invalid && field.$dirty
          }
        }
      },
      clearForm () {
        this.$v.$reset();
        this.form.firstName = null;
        this.form.lastName  = null;
        this.form.age       = null;
        this.form.gender    = null;
        this.form.email     = null;
      },
      saveUser () {
        this.sending = true

        // Instead of this timeout, here you can call your API
        window.setTimeout(() => {
          this.lastUser  = `${this.form.firstName} ${this.form.lastName}`;
          this.userSaved = true;
          this.sending   = false;
          this.rows.push({
            firstName: this.form.firstName,
            lastName: this.form.lastName,
            email: this.form.email,
            gender: this.form.gender,
            age: this.form.age
          })
          this.clearForm();
        }, 1000)
      },
      editUserData (id) {
        this.sending = true

        // Instead of this timeout, here you can call your API
        window.setTimeout(() => {
          this.lastUser  = `${this.form.firstName} ${this.form.lastName}`;
          this.userSaved = true;
          this.sending   = false;
          this.rows[id] = {
            firstName: this.form.firstName,
            lastName: this.form.lastName,
            email: this.form.email,
            gender: this.form.gender,
            age: this.form.age
          };
          this.clearForm();
          this.editUser = false;
          this.userId   = null;
        }, 1000)
      },
      validateUser (id = null) {
        this.$v.$touch();

        if (!this.$v.$invalid) {
          if(this.editUser){
            this.editUserData(id);
          } else {
            this.saveUser();
          }
        }
      },
      editRow (id) {
        this.$v.$reset();
        this.form.firstName = this.rows[id].firstName;
        this.form.lastName  = this.rows[id].lastName;
        this.form.age       = this.rows[id].age;
        this.form.gender    = this.rows[id].gender;
        this.form.email     = this.rows[id].email;

        this.editUser = true;
        this.userId   = id;
        window.scrollTo(0, 0);
      },
      deleteRow (id) {
        if(confirm("Are you sure delete this user?")){
          this.rows.splice(id, 1);
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  .md-progress-bar {
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
  }
  .md-card {
    margin: 0 auto;
  }
  .content {
    margin: 60px 0;
  }
</style>