<template>
    <div class="container">

        <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User Management</h3>

                <div class="card-tools">
                    <button class="btn btn-success " @click = "newModal">Add New <i class="fas fa-user-plus"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered at</th>
                    <th>Modify</th>
                  </tr>

                  <tr v-for="user in users" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.name |upText}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type}}</td>
                    <td>{{user.created_at | myDate}}</td>
                    <td>
                        <a href="#" @click="editModal(user)">
                            <i class="fa fa-edit"></i>   
                        </a>
                        /
                        <a href="#" @click="deleteUser(user.id)">
                            <i class="fa fa-trash red"></i>
                        </a>
                    </td>
                  </tr>
                  
                </tbody></table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
          <!-- Button trigger modal -->

<!-- Modal -->
    <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="addNewLabel" v-show="!editmode">Add User</h5>
                <h5 class="modal-title" id="addNewLabel" v-show="editmode">Update User</h5>

                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>
         <form @submit.prevent="editmode ? updateUser() : createUser()">  
            <div class="modal-body">

                <div class="form-group">
                    <input v-model="form.name" type="text" name="name" class="form-control" :class="{'is-invalid':form.errors.has('name')}"
                    placeholder="Name">
                    <has-error :form="form" field = "name"></has-error>
                </div>

                <div class="form-group">
                    <input v-model="form.email" type="email" name="email" class="form-control" :class="{'is-invalid':form.errors.has('email')}"
                    placeholder="Email Address">
                    <has-error :form="form" field = "email"></has-error>
                </div>
                
                <div class="form-group">
                    <textarea v-model="form.bio"  name="bio" class="form-control" :class="{'is-invalid':form.errors.has('bio')}"
                    placeholder="Bio">
                    </textarea>
                    <has-error :form="form" field = "bio"></has-error>
                </div>

                 <div class="form-group">
                    <select  v-model="form.type"  name="type" class="form-control" :class="{'is-invalid':form.errors.has('type')}"
                    placeholder="Bio">
                    <option value="">Select User Role</option>
                    <option value="admin">Admin</option>
                    <option value="user">Standart User</option>
                    <option value="author">Author</option>
                    </select>
                    <has-error :form="form" field = "type"></has-error>
                </div>
                
                <div class="form-group">
                    <input v-model="form.password" type="password" name="password" class="form-control" :class="{'is-invalid':form.errors.has('password')}"
                    placeholder="Password">
                    <has-error :form="form" field = "password"></has-error>
                </div>
              
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
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
        data(){
            return {
                editmode:false,
                users:{},
                form:new Form({
                    id :'',
                    name:'',
                    email:'',
                    password:'',
                    type:'',
                    bio:'',
                    photo:''
                })
            }
        },
        methods: {
            updateUser() {
                this.form.put('api/user/'+this.form.id)
                .then(()=>{
                    $('#addNew').modal('hide');
                     Fire.$emit('AfterCreate');  

                     Swal.fire(
                          'User Info Updated!',
                           'Your Data Has been Updated',
                            'success'
                             )
                     this.$Progress.finish();        

                })
                .catch(()=>{
                    this.$Progress.fail();
                });
            },
            editModal(user){
                this.editmode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);
            },
            newModal(){
                this.editmode = false;
                this.form.reset();
                $('#addNew').modal('show');
            },

            deleteUser(id){
                Swal.fire({
                        title: 'Are you sure?',
                        text: "You won't be able to revert this!",
                        type: 'warning',
                        showCancelButton: true,
                        confirmButtonColor: '#3085d6',
                        cancelButtonColor: '#d33',
                        confirmButtonText: 'Yes, delete it!'
                        }).then((result) => {
                            //send request to the server
                          
                            if (result.value){
                            this.form.delete('api/user/'+id).then(()=>{
                            Fire.$emit('AfterCreate');  

                              Swal.fire(
                               'Deleted!',
                               'Your Data Hase been Deleted',
                               'success'
                             )   
                                
                            }).catch(()=>{
                                Swal("Failed!","Something Wrong","Warning");
                            });
                            
                    }     
                })
            },
            loadUsers(){
                axios.get("api/user").then(({data})=>(this.users=data.data));
            },
            createUser(){
                this.$Progress.start();
                this.form.post('api/user');
                Fire.$emit('AfterCreate'); 
                $('#addNew').modal('hide')
                Toast.fire({
                type: 'success',
                title: 'User Created'
                })

                this.$Progress.finish();
            }        


        },
        created() {
            this.loadUsers();
            Fire.$on('AfterCreate',()=>{
                this.loadUsers();
            });
            //setInterval(()=>this.loadUsers(),3000);
        }
    }
</script>
