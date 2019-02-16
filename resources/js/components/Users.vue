<template>
    <div class="container">
        <div class="row pt-3" v-if="$gate.isAdminOrAuthor()">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users Table</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" @click="newModal">Add user <i class="fas fa-user-plus"></i></button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tbody>
                                <tr>
                                    <th>ID</th>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Type</th>
                                    <th>Registered At</th>
                                    <th>Modify</th>
                                </tr>
                                <tr v-for="user in users.data" :key="user.id">
                                    <td>{{ user.id }}</td>
                                    <td>{{ user.name }}</td>
                                    <td>{{ user.email }}</td>
                                    <td>{{ user.type | upText }}</td>
                                    <td>{{ user.created_at | myDate}}</td>
                                    <td>
                                        <a href="#" @click="editModal(user)"><i class="fa fa-edit"></i></a>
                                        /
                                        <a href="#" @click="deleteUser(user.id)">
                                            <i class="fa fa-trash text-danger"></i>
                                        </a>
                                    </td>
                                </tr>

                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                    <div class="card-footer">
                        <pagination :data="users" @pagination-change-page="getResults"></pagination>
                    </div>
                </div>
                <!-- /.card -->

            </div>
        </div>

        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>


        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNew" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-show="!editmode" class="modal-title" id="addNew">Add New</h5>
                        <h5 v-show="editmode" class="modal-title" id="addNew">Update User</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>

                    <form @submit.prevent="editmode ? updateUser() : createUser()">
                        <div class="modal-body">

                            <div class="form-group">
                                <label>Username</label>
                                <input v-model="form.name" type="text" name="name" class="form-control" :class="{ 'is-invalid': form.errors.has('name') }"
                                    placeholder="Name">
                                <has-error :form="form" field="name"></has-error>
                            </div>

                            <div class="form-group">
                                <label>Email</label>
                                <input v-model="form.email" type="email" name="email" class="form-control" :class="{ 'is-invalid': form.errors.has('email') }"
                                    placeholder="Email">
                                <has-error :form="form" field="email"></has-error>
                            </div>

                            <div class="form-group">
                                <label>Bio</label>
                                <textarea v-model="form.bio" name="bio" id="bio" class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"
                                    placeholder="Bio (Optional)"></textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>

                            <div class="form-group">
                                <label>Type</label>
                                <select v-model="form.type" name="type" id="type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                    <option value="">Select role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">User</option>
                                    <option value="author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>

                            <div class="form-group">
                                <label>Password</label>
                                <input v-model="form.password" type="password" name="password" id="password" class="form-control"
                                    :class="{ 'is-invalid': form.errors.has('password') }" placeholder="Password">
                                <has-error :form="form" field="password"></has-error>
                            </div>

                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                            <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
                            <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
                        </div>
                    </form>

                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {

        data() {
            return {
                editmode: false,

                users: {},

                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },

        methods: {

           getResults(page = 1) {
			axios.get('api/user?page=' + page)
				.then(response => {
					this.users = response.data;
                });
           },

            updateUser() {
                this.$Progress.start();
                this.form.put('api/user/' + this.form.id)
                    .then(() => {
                        //success
                        $('#addNew').modal('hide')
                        swal.fire(
                            'Updated!',
                            'Your file has been updated.',
                            'success'
                        )
                        this.$Progress.finish();
                        Fire.$emit('AfterCreated');
                    })
                    .catch(() => {
                        this.$Progress.fail();
                    })
            },

            editModal(user) {
                this.editmode = true;
                this.form.reset();
                $('#addNew').modal('show')
                this.form.fill(user);
            },

            newModal() {
                this.editmode = false;
                this.form.reset();
                $('#addNew').modal('show')
            },

            deleteUser(id) {
                swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if (result.value) {
                        //request to server
                        this.form.delete('api/user/' + id).then(() => {

                            swal.fire(
                                'Deleted!',
                                'Your file has been deleted.',
                                'success'
                            );

                            Fire.$emit('AfterCreated');

                        }).catch(() => {
                            swal.fire(
                                'Failed to Delete!',
                                'Something went wrong.',
                                'warning'
                            )
                        });
                    }

                })
            },

            loadUsers() {

                if (this.$gate.isAdminOrAuthor()) {

                    axios.get("api/user").then(({
                        data
                    }) => (this.users = data));

                }
            },
            createUser() {
                this.$Progress.start();
                this.form.post('api/user')

                    .then(() => {
                        Fire.$emit('AfterCreated');

                        $('#addNew').modal('hide')
                        toast.fire({
                            type: 'success',
                            title: 'Signed in successfully'
                        })

                        this.$Progress.finish();
                    })

                    .catch(() => {

                    })

            }
        },

        created() {
            Fire.$on('searching', () => {
                let query = this.$parent.search;
                axios.get('api/findUser?q=' + query)
                .then((data) => {
                    this.users = data.data
                })
                .catch(() => {

                })
            });
            this.loadUsers();
            Fire.$on('AfterCreated', () => {
                this.loadUsers();
            });
            //setInterval(()=>this.loadUsers(), 3000);
        }
    }

</script>
