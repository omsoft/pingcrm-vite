<template>
  <div>
    <h1 class="mb-8 font-bold text-3xl">
      Users
    </h1>
    <div class="mb-6 flex justify-between items-center">
      <search-filter
        v-model="form.search"
        class="w-full max-w-md mr-4"
        @reset="reset"
      >
        <label
          class="block text-gray-800"
          for="role-filter"
        >Role:</label>
        <select
          id="role-filter"
          v-model="form.role"
          class="mt-1 w-full form-select"
        >
          <option :value="null"/>
          <option value="user">
            User
          </option>
          <option value="owner">
            Owner
          </option>
        </select>
        <label
          class="mt-4 block text-gray-800"
          for="trashed-filter"
        >Trashed:</label>
        <select
          id="trashed-filter"
          v-model="form.trashed"
          class="mt-1 w-full form-select"
        >
          <option :value="null"/>
          <option value="with">
            With Trashed
          </option>
          <option value="only">
            Only Trashed
          </option>
        </select>
      </search-filter>
      <inertia-link
        v-if="can.create_user"
        class="btn-indigo"
        :href="$api.users.new.path()"
      >
        Create <span class="hidden md:inline">User</span>
      </inertia-link>
    </div>
    <div class="bg-white rounded shadow overflow-x-auto">
      <table class="w-full whitespace-nowrap">
        <thead>
          <tr class="text-left font-bold">
            <th class="px-6 pt-6 pb-4">
              Name
            </th>
            <th class="px-6 pt-6 pb-4">
              Email
            </th>
            <th
              class="px-6 pt-6 pb-4"
              colspan="2"
            >
              Role
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="user in users"
            :key="user.id"
            class="hover:bg-gray-100 focus-within:bg-gray-100"
          >
            <td class="border-t">
              <inertia-link
                class="px-6 py-4 flex items-center focus:text-indigo-500"
                :href="pathToEdit(user)"
                aria-label="Edit"
              >
                <img
                  v-if="user.photo"
                  class="block w-5 h-5 rounded-full mr-2 -my-2"
                  :src="user.photo"
                  alt="Photo"
                >
                {{ user.name }}
                <icon
                  v-if="user.deleted_at"
                  name="trash"
                  class="flex-shrink-0 w-3 h-3 fill-gray-500 ml-2"
                />
              </inertia-link>
            </td>
            <td class="border-t">
              <inertia-link
                class="px-6 py-4 flex items-center"
                :href="pathToEdit(user)"
                tabindex="-1"
                aria-label="Edit"
              >
                {{ user.email }}
              </inertia-link>
            </td>
            <td class="border-t">
              <inertia-link
                class="px-6 py-4 flex items-center"
                :href="pathToEdit(user)"
                tabindex="-1"
                aria-label="Edit"
              >
                {{ user.owner ? 'Owner' : 'User' }}
              </inertia-link>
            </td>
            <td class="border-t w-px">
              <inertia-link
                class="px-4 flex items-center"
                :href="pathToEdit(user)"
                tabindex="-1"
                aria-label="Edit"
              >
                <icon
                  name="cheveron-right"
                  class="block w-6 h-6 fill-gray-500"
                />
              </inertia-link>
            </td>
          </tr>
          <tr v-if="users.length === 0">
            <td
              class="border-t px-6 py-4"
              colspan="4"
            >
              No users found.
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import Icon from '@/Shared/Icon.vue'
import Layout from '@/Layouts/Main.vue'
import mapValues from 'lodash/mapValues'
import pickBy from 'lodash/pickBy'
import SearchFilter from '@/Shared/SearchFilter.vue'
import throttle from 'lodash/throttle'
import UsersApi from '@/api/UsersApi'

export default {
  metaInfo: { title: 'Users' },
  components: {
    Icon,
    SearchFilter,
  },
  layout: Layout,
  props: {
    users: {
      type: Array,
      required: true,
    },
    filters: {
      type: Object,
      required: true,
    },
    can: {
      type: Object,
      required: true,
    },
  },
  data () {
    return {
      form: {
        search: this.filters.search,
        role: this.filters.role,
        trashed: this.filters.trashed,
      },
    }
  },
  watch: {
    form: {
      handler: throttle(function () {
        const query = pickBy(this.form)
        UsersApi.list({
          query: Object.keys(query).length ? query : { remember: 'forget' },
          preserveState: true,
          preserveScroll: true,
          replace: true,
          only: ['users'],
        })
      }, 150),
      deep: true,
    },
  },
  methods: {
    pathToEdit (user) {
      return UsersApi.edit.path(user)
    },
    reset () {
      this.form = mapValues(this.form, () => null)
    },
  },
}
</script>
