<template>
  <div class="h-100">
    <v-toolbar flat height="38" class="toolbar-border-bottom">
      <v-text-field
        v-model="query"
        style="max-width: 300px"
        dense
        solo
        flat
        class="search-field caption"
        hide-details
        placeholder="Filter by email"
        @keypress.enter="loadUsers"
      >
        <template #prepend-inner>
          <v-icon class="mt-1" small>
            search
          </v-icon>
        </template>
      </v-text-field>
      <v-spacer />

      <!-- tooltip="Reload roles" -->
      <x-btn
        v-ge="['roles','reload']"
        outlined
        :tooltip="$t('activity.reloadRoles')"
        color="primary"
        small
        :disabled="loading"
        @click="loadUsers"
        @click.prevent
      >
        <v-icon small left>
          refresh
        </v-icon>
        <!-- Reload -->
        {{ $t('general.reload') }}
      </x-btn>
      <!-- tooltip="Add new role" -->
      <x-btn
        v-if="_isUIAllowed('newUser')"
        v-ge="['roles','add new']"
        outlined
        :tooltip="$t('tooltip.addRole')"
        color="primary"
        small
        :disabled="loading"
        @click="addUser"
      >
        <v-icon small left>
          mdi-plus
        </v-icon>
        <!-- New User -->
        {{ $t('activity.newUser') }}
      </x-btn>
    </v-toolbar>

    <v-card style="height:calc(100% - 38px)">
      <v-container style="height: 100%" fluid>
        <!--          <div class="d-flex d-100 justify-center">-->

        <v-row style="height:100%">
          <v-col cols="12" class="h-100">
            <v-card class="h-100 elevation-0">
              <v-row style="height:100%">
                <v-col offset="2" :cols="8" class="h-100" style="overflow-y: auto">
                  <!--                  <v-card class="h-100 px-4 py-2">-->

                  <!--                <v-row>
                                    <v-col>

                                    </v-col>
                                    <v-col class="flex-shrink-1 flex-grow-0"><h4 class="text-center text-capitalize mt-2 d-100"
                                                                                 style="min-width:100px">User List</h4></v-col>
                                    <v-col></v-col>

                                  </v-row>-->
                  <v-data-table
                    v-if="users"
                    dense
                    :headers="[{},{},{},{}]"
                    hide-default-header
                    :hide-default-footer="count < limit"
                    :options.sync="options"
                    :items="users"
                    :items-per-page.sync="limit"
                    :server-items-length="count"
                  >
                    <template #header>
                      <thead>
                        <tr class="text-left">
                          <!--                        <th>#</th>-->
                          <th class="font-weight-regular caption">
                            <v-icon small>
                              mdi-email-outline
                            </v-icon>
                            <!-- Email -->
                            {{ $t('labels.email') }}
                          </th>
                          <th class="font-weight-regular caption">
                            <v-icon small>
                              mdi-drama-masks
                            </v-icon>
                            <!-- Roles -->
                            {{ $t('objects.roles') }}
                          </th>
                          <th class="font-weight-regular caption">
                            <!--                          <v-icon small class="mt-n1">mdi-cursor-default-outline</v-icon>-->
                            <!-- Actions -->
                            {{ $t('labels.actions') }}
                          </th>
                        </tr>
                      </thead>
                    </template>

                    <template #item="{item}">
                      <tr @click="selectedUser = item">
                        <!--          <td>
                                    <v-radio-group dense hide-details v-model="selectedUserIndex" class="mt-n2">
                                      <v-radio :value="index"
                                      ></v-radio>
                                    </v-radio-group>
                                  </td>-->
                        <td>{{ item.email }}</td>
                        <td>
                          <!--                          {{ item.roles }}-->

                          <v-chip
                            v-if="item.roles"
                            class="mr-1"
                            :color="rolesColors[getRole(item.roles)]"
                          >
                            {{ getRole(item.roles) }}
                          </v-chip>

                          <!--                    <v-edit-dialog-->
                          <!--                    >-->
                          <!--                      <div-->
                          <!--                        :title="item.roles"-->
                          <!--                        style="width:180px;overflow:hidden;white-space: nowrap;text-overflow:ellipsis"> {{-->
                          <!--                          item.roles-->
                          <!--                        }}-->
                          <!--                      </div>-->
                          <!--                      <template v-slot:input>-->
                          <!--                        <v-text-field-->
                          <!--                          v-model="item.roles"-->
                          <!--                          label="Edit"-->
                          <!--                          single-line-->
                          <!--                        ></v-text-field>-->
                          <!--                      </template>-->
                          <!--                    </v-edit-dialog>-->
                        </td>
                        <td>
                          <!-- tooltip="Edit User" -->
                          <x-icon
                            v-if="item.project_id"
                            :tooltip="$t('activity.editUser')"
                            icon-class=""
                            color="primary"
                            small
                            @click.prevent.stop="invite_token = null; selectedUser = item; userEditDialog = true"
                          >
                            mdi-pencil-outline
                          </x-icon>
                          <span v-if="!item.project_id">
                            <x-icon
                              tooltip="Add user to project"
                              color="primary"
                              small
                              @click="inviteUser(item.email)"
                            >
                              mdi-plus
                            </x-icon>
                            <!-- <x-icon
                              tooltip="Remove user from NocoDB"
                              class="ml-2"
                              color="error"
                              small
                              @click.prevent.stop="deleteId = item.id; deleteItem = item.id;showConfirmDlg = true;deleteUserType='DELETE_FROM_NOCODB'"
                            >
                              mdi-delete-forever-outline
                            </x-icon> -->
                          </span>
                          <!-- tooltip="Remove user from project" -->
                          <x-icon
                            v-else
                            :tooltip="$t('activity.deleteUser')"
                            class="ml-2"
                            color="error"
                            small
                            @click.prevent.stop="deleteId = item.id; deleteItem = item.id;showConfirmDlg = true;deleteUserType='DELETE_FROM_PROJECT'"
                          >
                            mdi-delete-outline
                          </x-icon>

                          <!-- tooltip="Resend invite email" -->
                          <x-icon
                            v-if="item.invite_token"
                            :tooltip="$t('activity.resendInvite')"
                            icon-class="mt-n1"
                            color="primary"
                            small
                            @click.prevent.stop="rensendInvite(item.id)"
                          >
                            mdi-email-send-outline
                          </x-icon>

                          <!-- tooltip="Copy invite url" -->
                          <x-icon
                            v-if="item.invite_token"
                            :tooltip="$t('activity.copyInviteURL')"
                            icon-class=""
                            color="primary"
                            small
                            @click.prevent.stop="clipboard(getInviteUrl(item.invite_token)); $toast.success('Invite url copied to clipboard').goAway(3000)"
                          >
                            mdi-content-copy
                          </x-icon>
                        </td>
                      </tr>
                    </template>
                  </v-data-table>
                  <!-- tooltip="Add new user" -->
                  <div class="mt-10 text-center">
                    <x-btn
                      v-if="_isUIAllowed('newUser')"
                      v-ge="['roles','add new']"
                      outlined
                      :tooltip="$t('msg.info.addUser')"
                      color="primary"
                      small
                      :disabled="loading"
                      @click="addUser"
                    >
                      <v-icon small left>
                        mdi-plus
                      </v-icon>
                      <!-- New User -->
                      {{ $t('activity.newUser') }}
                    </x-btn>
                  </div>

                  <feedback-form class="mx-auto mt-6" />
                </v-col>
              </v-row>
            </v-card>
          </v-col>
        </v-row>

        <table v-if="false" class="mx-auto users-table" style="min-width:700px">
          <thead>
            <tr>
              <th />
              <th>Email</th>
              <th>Roles</th>
              <th />
            </tr>
          </thead>
          <tbody>
            <tr v-for="user in users" :key="user.email">
              <td>
                <v-icon x-large>
                  mdi-account-outline
                </v-icon>
              </td>
              <td class="px-1 py-1" align="top">
                <v-text-field
                  solo
                  class="elevation-0"
                  disabled
                  :value="user.email"
                  hide-details
                  dense
                />
              </td>
              <td class="px-1 py-1">
                <set-list-checkbox-cell v-model="user.roles" :values="roles" />
              </td>
              <td align="middle">
                <v-icon large>
                  mdi-close
                </v-icon>
                <v-icon large>
                  mdi-save
                </v-icon>
              </td>
            </tr>
            <tr>
              <td>
                <v-icon x-large>
                  mdi-account-outline
                </v-icon>
              </td>
              <td class="px-1 py-1" align="top">
                <v-text-field
                  solo
                  class="elevation-0"

                  hide-details
                  dense
                />
              </td>
              <td class="px-1 py-1">
                <set-list-checkbox-cell :values="roles" />
              </td>
              <td align="middle">
                <v-icon large>
                  mdi-account-plus
                </v-icon>
              </td>
            </tr>
          </tbody>
        </table>
      </v-container>
    </v-card>

    <dlg-label-submit-cancel
      type="primary"
      :actions-mtd="confirmDelete"
      :heading="dialogMessage"
      :dialog-show="showConfirmDlg"
    />

    <!-- todo: move to a separate component-->
    <v-dialog v-model="userEditDialog" :width="invite_token ? 700 :700" @close="invite_token = null">
      <v-card v-if="selectedUser" style="min-height: 100%">
        <v-card-title>
          {{ $t('activity.share') }} : {{ $store.getters['project/GtrProjectName'] }}
          <!--
          <h4 class="text-center text-capitalize mt-2 d-100 display-1">
            <template v-if="invite_token">
              Copy Invite Token
            </template>
            <template v-else-if="selectedUser.id">
              Edit User
            </template>
            <template v-else>
              Invite
            </template>
          </h4>-->

          <div class="nc-header-border" />
        </v-card-title>

        <v-card-text>
          <div>
            <v-icon small>
              mdi-account-outline
            </v-icon>
            <template v-if="invite_token">
              Copy Invite Token
            </template>
            <template v-else-if="selectedUser.id">
              Edit User
            </template>
            <template v-else>
              <!-- Invite Team -->
              {{ $t('activity.inviteTeam') }}
            </template>
          </div>
          <div class="pa-4 nc-invite-container">
            <div v-if="invite_token" class="mt-6  align-center">
              <v-alert
                v-ripple
                type="success"
                outlined

                class="pointer"
                @click="clipboard(inviteUrl); $toast.success('Copied invite url to clipboard').goAway(3000)"
              >
                <template #append>
                  <v-icon color="green" class="ml-2">
                    mdi-content-copy
                  </v-icon>
                </template>
                <div class="ellipsis d-100">
                  {{ inviteUrl }}
                </div>
              </v-alert>

              <p class="caption grey--text mt-3">
                {{ $t('msg.info.userInviteNoSMTP') }}
                <!-- Looks like you have not configured mailer yet! <br>Please copy above -->
                <!-- invite -->
                <!-- link and send it to -->
                {{ invite_token && (invite_token.email || invite_token.emails && invite_token.emails.join(', ')) }}.
              </p>

              <div class="text-right">
                <!--tooltip="Invite more users"-->
                <x-btn
                  :tooltip="$t('tooltip.inviteMore')"
                  small
                  outlined
                  btn.class="grey--text"
                  @click="invite_token = null, selectedUser = {}"
                >
                  <v-icon small color="grey" class="mr-1">
                    mdi-account-multiple-plus-outline
                  </v-icon>
                  <!--Invite more-->
                  {{ $t('activity.inviteMore') }}
                </x-btn>
              </div>

              <!--          todo: show error message if failed-->
            </div>
            <template v-else>
              <v-form ref="form" v-model="valid" @submit.prevent="saveUser">
                <v-row class="my-0">
                  <v-col cols="8" class="py-0">
                    <!--hint="You can add multiple comma(,) separated emails"-->
                    <v-text-field
                      ref="email"
                      v-model="selectedUser.email"
                      :disabled="!!selectedUser.id"
                      dense
                      validate-on-blur
                      outlined
                      :rules="validate && emailRules"
                      class="caption"
                      :hint="$t('msg.info.addMultipleUsers')"
                      label="Email"
                      @input="edited=true"
                    >
                      <template #label>
                        <span class="caption">
                          <!-- Email -->
                          {{ $t('labels.email') }}
                        </span>
                      </template>
                    </v-text-field>
                  </v-col>
                  <v-col cols="4" class="py-0">
                    <!--label="Select User Role"-->
                    <v-combobox
                      v-model="selectedRoles"
                      outlined
                      class="role-select caption"
                      hide-details="auto"
                      :items="roles"
                      :label="$t('labels.selectUserRole')"
                      dense
                      deletable-chips
                      @change="edited = true"
                    >
                      <template #selection="{item}">
                        <v-chip small :color="rolesColors[item]">
                          {{ item }}
                        </v-chip>
                      </template>
                      <template #item="{item}">
                        <div>
                          <div>{{ item }}</div>
                          <div class="mb-2 caption grey--text">
                            {{ roleDescriptions[item] }}
                          </div>
                        </div>
                      </template>
                    </v-combobox>
                  </v-col>
                </v-row>
              </v-form>
              <!--        </v-card-text>
        <v-card-actions class="justify-center">-->

              <!-- tooltip="Save Changes" -->
              <div class="text-center mt-0">
                <x-btn
                  v-ge="['rows','save']"
                  :tooltip="$t('tooltip.saveChanges')"
                  color="primary"

                  btn.class="nc-invite-or-save-btn"
                  @click="saveUser"
                >
                  <v-icon small left>
                    {{ selectedUser.id ? 'save' : 'mdi-send' }}
                  </v-icon>
                  {{ selectedUser.id ? $t('general.save') : $t('activity.invite') }}
                </x-btn>
              </div>
            </template>
            <!--        </v-card-actions>-->
          </div>
        </v-card-text>
        <v-card-text>
          <share-base />
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import FeedbackForm from '@/components/feedbackForm'
import SetListCheckboxCell from '@/components/project/spreadsheet/components/editableCell/setListCheckboxCell'
import { enumColor } from '@/components/project/spreadsheet/helpers/colors'
import DlgLabelSubmitCancel from '@/components/utils/dlgLabelSubmitCancel'
import { isEmail } from '@/helpers'
import ShareBase from '~/components/base/shareBase'
import XBtn from '~/components/global/xBtn'

export default {
  name: 'UserManagement',
  components: { XBtn, ShareBase, FeedbackForm, DlgLabelSubmitCancel, SetListCheckboxCell },
  data: () => ({
    validate: false,
    deleteItem: null,
    invite_token: null,
    userEditDialog: false,
    limit: 10,
    showConfirmDlg: false,
    users: null,
    count: 0,
    options: {},
    loading: false,
    selectedUser: null,
    roles: [],
    query: '',
    deleteId: null,
    edited: false,
    valid: null,
    emailRules: [
      v => !!v || 'E-mail is required',
      (v) => {
        const invalidEmails = (v || '').split(/\s*,\s*/).filter(e => !isEmail(e))
        return !invalidEmails.length || `"${invalidEmails.join(', ')}" - invalid email`
      }
    ],
    userList: [],
    roleDescriptions: {},
    deleteUserType: '' // [DELETE_FROM_PROJECT, DELETE_FROM_NOCODB]
  }),
  computed: {
    roleNames() {
      return this.roles.map(r => r.title)
    },
    inviteUrl() {
      return this.invite_token ? `${location.origin}${location.pathname}#/user/authentication/signup/${this.invite_token.invite_token}` : null
    },
    rolesColors() {
      const colors = this.$store.state.windows.darkTheme ? enumColor.dark : enumColor.light
      return this.roles.reduce((o, r, i) => {
        o[r] = colors[i % colors.length]
        return o
      }, {})
    },
    selectedRoles: {
      get() {
        return (this.selectedUser && this.selectedUser.roles ? this.selectedUser.roles.split(',') : []).sort((a, b) => this.roleNames.indexOf(a) - this.roleNames.indexOf(a))[0]
      },
      set(roles) {
        if (this.selectedUser) {
          this.selectedUser.roles = roles // .filter(Boolean).join(',')
        }
      }
    },
    selectedUserIndex: {
      get() {
        return this.users ? this.users.findIndex(u => u.email === this.selectedUser.email) : -1
      },
      set(i) {
        this.selectedUser = this.users[i]
      }
    },
    dialogMessage() {
      let msg = 'Do you want to remove the user'
      if (this.deleteUserType === 'DELETE_FROM_PROJECT') { msg += ' from Project' } else if (this.deleteUserType === 'DELETE_FROM_NOCODB') { msg += ' from NocoDB' }
      msg += '?'
      return msg
    }
  },
  watch: {
    options: {
      async handler() {
        await this.loadUsers()
      },
      deep: true
    },
    userEditDialog(v) {
      // if (!v) { this.validate = false }
      if (v && (this.selectedUser && !this.selectedUser.id)) {
        this.$nextTick(() => {
          setTimeout(() => {
            this.$refs.email.$el.querySelector('input').focus()
          }, 100)
        })
      }
    }
  },
  async created() {
    this.$eventBus.$on('show-add-user', this.addUser)
    await this.loadUsers()
    await this.loadRoles()
  },
  beforeDestroy() {
    this.$eventBus.$off('show-add-user', this.addUser)
  },
  methods: {
    getRole(roles) {
      return (roles ? roles.split(',') : []).sort((a, b) => this.roleNames.indexOf(a) - this.roleNames.indexOf(a))[0]
    },
    simpleAnim() {
      const count = 30
      const defaults = {
        origin: { y: 0.7 },
        zIndex: 9999999
      }

      function fire(particleRatio, opts) {
        window.confetti(Object.assign({}, defaults, opts, {
          particleCount: Math.floor(count * particleRatio)
        }))
      }

      fire(0.25, {
        spread: 26,
        startVelocity: 55
      })
      fire(0.2, {
        spread: 60
      })
      fire(0.35, {
        spread: 100,
        decay: 0.91,
        scalar: 0.8
      })
      fire(0.1, {
        spread: 120,
        startVelocity: 25,
        decay: 0.92,
        scalar: 1.2
      })
      fire(0.1, {
        spread: 120,
        startVelocity: 45
      })
    },
    getInviteUrl(token) {
      return token ? `${location.origin}${location.pathname}#/user/authentication/signup/${token}` : null
    },

    clipboard(str) {
      const el = document.createElement('textarea')
      el.addEventListener('focusin', e => e.stopPropagation())
      el.value = str
      document.body.appendChild(el)
      el.select()
      document.execCommand('copy')
      document.body.removeChild(el)
    },
    async rensendInvite(id) {
      try {
        await this.$axios.post('/admin/resendInvite/' + id, {
          projectName: this.$store.getters['project/GtrProjectName']
        }, {
          headers: {
            'xc-auth': this.$store.state.users.token
          },
          params: {
            project_id: this.$route.params.project_id
          }
        })
        this.$toast.success('Invite email sent successfully').goAway(3000)
        await this.loadUsers()
      } catch (e) {
        this.$toast.error(e.response.data.msg).goAway(3000)
      }
    },
    async loadUsers() {
      try {
        const { page = 1, itemsPerPage = 20 } = this.options
        const data = (await this.$axios.get('/admin', {
          headers: {
            'xc-auth': this.$store.state.users.token
          },
          params: {
            limit: itemsPerPage,
            offset: (page - 1) * itemsPerPage,
            query: this.query,
            project_id: this.$route.params.project_id
          }
        })).data
        this.count = data.count
        this.users = data.list
        if (!this.selectedUser && this.users && this.users[0]) {
          this.selectedUserIndex = 0
        }
      } catch (e) {
        console.log(e)
      }
    },
    async loadRoles() {
      try {
        this.roles = (await this.$axios.get('/admin/roles', {
          headers: {
            'xc-auth': this.$store.state.users.token
          },
          params: {
            project_id: this.$route.params.project_id
          }
        })).data.map((role) => {
          this.roleDescriptions[role.title] = role.description
          return role.title
        }).filter(role => role !== 'guest')
      } catch (e) {
        console.log(e)
      }
    },
    async deleteUser(id, type) {
      try {
        await this.$axios.delete('/admin/' + id, {
          params: {
            project_id: this.$route.params.project_id,
            email: this.deleteItem.email,
            type
          },
          headers: {
            'xc-auth': this.$store.state.users.token
          }
        })
        this.$toast.success(`Successfully removed the user from ${type === 'DELETE_FROM_PROJECT' ? 'project' : 'NocoDB'}`).goAway(3000)
        await this.loadUsers()
      } catch (e) {
        this.$toast.error(e.response.data.msg).goAway(3000)
      }
    },
    async confirmDelete(hideDialog) {
      if (hideDialog) {
        this.showConfirmDlg = false
        return
      }
      await this.deleteUser(this.deleteId, this.deleteUserType)
      this.showConfirmDlg = false
    },
    addUser() {
      this.invite_token = null
      this.selectedUser = {
        roles: 'editor'
      }
      this.userEditDialog = true
    },
    async inviteUser(email) {
      try {
        await this.$axios.post('/admin', {
          email,
          project_id: this.$route.params.project_id,
          projectName: this.$store.getters['project/GtrProjectName']
        }, {
          headers: {
            'xc-auth': this.$store.state.users.token
          }
        })
        this.$toast.success('Successfully added user to project').goAway(3000)
        await this.loadUsers()
      } catch (e) {
        this.$toast.error(e.response.data.msg).goAway(3000)
      }
    },
    async saveUser() {
      this.validate = true
      await this.$nextTick()
      if (this.loading || !this.$refs.form.validate() || !this.selectedUser) {
        return
      }

      if (!this.edited) {
        this.userEditDialog = false
      }

      try {
        let data
        if (this.selectedUser.id) {
          await this.$axios.put('/admin/' + this.selectedUser.id, {
            roles: this.selectedUser.roles,
            email: this.selectedUser.email,
            project_id: this.$route.params.project_id,
            projectName: this.$store.getters['project/GtrProjectName']
          }, {
            headers: {
              'xc-auth': this.$store.state.users.token
            }
          })
        } else {
          data = await this.$axios.post('/admin', {
            ...this.selectedUser,
            project_id: this.$route.params.project_id,
            projectName: this.$store.getters['project/GtrProjectName']
          }, {
            headers: {
              'xc-auth': this.$store.state.users.token
            }
          })
        }
        this.$toast.success('Successfully updated the user details').goAway(3000)
        await this.loadUsers()
        if (data && data.data && data.data.invite_token) {
          this.invite_token = data.data
          this.simpleAnim()
          return
        }
      } catch (e) {
        this.$toast.error(e.response.data.msg).goAway(3000)
      }

      this.userEditDialog = false
      await this.loadUsers()
    }
  }
}
</script>

<style scoped lang="scss">
::v-deep {
  .v-alert__wrapper > .v-icon {
    align-self: center;
  }

  tbody tr:nth-of-type(odd) {
    background-color: transparent;
  }

  .search-field.v-text-field > .v-input__control, .search-field.v-text-field > .v-input__control > .v-input__slot {
    min-height: auto;
  }

  .role-select {
    .v-select__selections {
      //padding-bottom: 0 !important;
      min-height: auto !important;

      .v-chip {
        margin-top: 6px;
        margin-bottom: 2px;
      }
    }
  }
}

.users-table {
  td:first-child {
    width: 50px
  }

  td:last-child {
    width: 50px
  }
}

::v-deep .nc-invite-container {
  border-radius: 4px;
  border: 2px solid var(--v-backgroundColor-base);
  background: var(--v-backgroundColor-base);

  .v-input .v-input__slot {

    background: var(--v-backgroundColorDefault-base);
  }
}

.nc-header-border {
  width: 100%;
  border-bottom: 2px solid var(--v-backgroundColor-base);
  margin-bottom: 10px;
}
</style>
<!--
/**
 * @copyright Copyright (c) 2021, Xgene Cloud Ltd
 *
 * @author Naveen MR <oof1lab@gmail.com>
 * @author Pranav C Balan <pranavxc@gmail.com>
 * @author Wing-Kam Wong <wingkwong.code@gmail.com>
 *
 * @license GNU AGPL version 3 or any later version
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU Affero General Public License as
 * published by the Free Software Foundation, either version 3 of the
 * License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU Affero General Public License for more details.
 *
 * You should have received a copy of the GNU Affero General Public License
 * along with this program. If not, see <http://www.gnu.org/licenses/>.
 *
 */
-->
