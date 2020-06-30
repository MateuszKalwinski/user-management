<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users List</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" @click="newModal">Add new
                                <i class="fas fa-user-plus fa-fw"></i>
                            </button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover text-nowrap">
                            <thead>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Type</th>
                                <th>Registered at</th>
                            </tr>
                            </thead>
                            <tbody>
                            <tr v-for="user in users.data" :key="user.id">
                                <td>{{user.id}}</td>
                                <td>{{user.name}}</td>
                                <td>{{user.email}}</td>
                                <td>{{user.type | upText}}</td>
                                <td>{{user.created_at | shortDate}}</td>
                                <td>
                                    <button type="button" class="btn-simple" @click="editModal(user)">
                                        <i class="fas fa-edit text-blue"></i>
                                    </button>
                                    /
                                    <button type="button" class="btn-simple" @click="deleteUser(user.id)">
                                        <i class="fas fa-trash text-red"></i>
                                    </button>
                                </td>
                            </tr>


                            </tbody>
                        </table>
                    </div>
                    <div class="card-footer">
                        <pagination :data="users" @pagination-change-page="getResults"></pagination>
                    </div>
                    <!-- /.card-body -->
                </div>
                <!-- /.card -->
            </div>
        </div>

        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>

        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel"
             aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="addNewLabel">{{modalTitle}}</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode? updateUser() : createUser()">
                        <div class="modal-body">
                            <div class="form-group">
                                <input v-model="form.name" type="text" name="name"
                                       placeholder="Name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>

                            <div class="form-group">
                                <input v-model="form.email" type="text" name="email"
                                       placeholder="Email address"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">
                            <textarea v-model="form.bio" type="text" name="bio"
                                      placeholder="Short bio for user (optional)"
                                      class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>
                            <div class="form-group">
                                <select v-model="form.type" type="text" name="type"
                                        placeholder="Short bio for user (optional)"
                                        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                    <option value="">Select user role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">User</option>
                                    <option value="author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>

                            <div class="form-group">
                                <input v-model="form.password" type="password" name="password"
                                       placeholder="Password"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>

                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                            <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
                            <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>

                        </div>
                    </form>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
    import Form from 'vform'

    export default {
        data() {
            return {
                editMode: false,
                users: {},
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: '',
                }),
                modalTitle: '',
            }
        },
        methods:{

            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
            },
            editModal(user){
                this.editMode = true;
                this.form.reset();
                this.form.fill(user);
                this.modalTitle = 'Edit user'
                $('#addNew').modal('show');
            },
            newModal(){
                this.editMode = false;
                this.form.reset();
                this.modalTitle = 'Add new';
                $('#addNew').modal('show');
            },
            loadUsers(){
                if (this.$gate.isAdminOrAuthor()){
                    axios.get('api/user').then(({data}) => (this.users = data));
                }
            },
            createUser(){
                this.$Progress.start();
                this.form.post('/api/user')
                .then(() => {
                    Fire.$emit('AfterCreate');
                    $('#addNew').modal('hide');
                    toast.fire({
                        icon: 'success',
                        title: 'User created in successfully'
                    })

                    this.$Progress.finish()
                })
                .catch(() => {
                    this.$Progress.fail();
                })
            },
            updateUser(){
                this.$Progress.start();
                this.form.put('api/user/'+this.form.id)
                .then(() => {
                    Fire.$emit('AfterCreate');
                    $('#addNew').modal('hide');
                    toast.fire({
                        icon: 'success',
                        title: 'User updated in successfully'
                    })
                    this.$Progress.finish()
                })
                .catch(() => {
                    this.$Progress.fail();
                })
            },
            deleteUser(id){
                swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if (result.value) {
                        this.form.delete('/api/user/' + id)
                            .then(() => {
                                swal.fire(
                                    'Deleted!',
                                    'User has been deleted.',
                                    'success'
                                )
                                Fire.$emit('AfterCreate');
                            })
                            .catch(() => {
                                swal.fire('Failed!', 'There was something wrong', 'warning');
                            })
                    }
                })
            }
        },
        created() {
            this.loadUsers();
            Fire.$on('AfterCreate', () => {
                this.loadUsers();
            })
        }
    }
</script>
<style>
    .btn-simple{
        border: none;
        background: none;
    }
</style>
