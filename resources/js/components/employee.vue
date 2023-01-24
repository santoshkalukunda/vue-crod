<template>
    <!-- Button trigger modal -->
    <button type="button" class="btn btn-primary fload-right" v-on:click="formReset" data-bs-toggle="modal" data-bs-target="#exampleModal">
        New Employee
    </button>

    <!-- Modal -->
    <div class="modal fade" id="exampleModal">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header">
                    <h1 class="modal-title fs-5" id="exampleModalLabel">New Employee</h1>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <form>
                        <div class="text-sm text-danger" if="errors != ''">
                            <p v-for="error in errors" :key="error">
                                <small>
                                    {{ error }}
                                </small>
                            </p>
                        </div>
                        <div class="mb-3">
                            <label for="exampleFormControlInput1" class="form-label">Name</label>
                            <input type="text" v-model="form.name" class="form-control" id="exampleFormControlInput1"
                                placeholder="name" autofocus>
                        </div>
                        <div class="mb-3">
                            <label for="exampleFormControlInput1" class="form-label">Email address</label>
                            <input type="email" v-model="form.email" class="form-control"
                                id="exampleFormControlInput1" placeholder="name@example.com">
                        </div>
                        <div class="mb-3">
                            <label for="exampleFormControlInput1" class="form-label">Phone</label>
                            <input type="text" v-model="form.phone" class="form-control"
                                id="exampleFormControlInput1" placeholder="Phone">
                        </div>
                        <button type="button" v-if="employee_id == ''" class="btn btn-primary mx-2" v-on:click="storeEmployee">Save</button>
                        <button type="button" v-else class="btn btn-primary mx-2" v-on:click="updateEmployee">Update</button>
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                    </form>
                </div>
            </div>
        </div>
    </div>
    <table class="table table-border table-h">
        <thead>
            <tr>
                <th>Name</th>
                <th>email</th>
                <th>phone</th>
                <th>action</th>
            </tr>
        </thead>
        <tbody>
            <template v-for="employee in employees " :key="employee.id">
                <tr>
                    <td>{{ employee . name }}</td>
                    <td>{{ employee . email }}</td>
                    <td>{{ employee . phone }}</td>
                    <td>
                        <button type="button" class="btn btn-primary fload-right" data-bs-toggle="modal"
                            data-bs-target="#exampleModal" @click="editEmployee(employee)">
                            Edit
                        </button>
                     
                        <button type="button" class="btn btn-danger mx-2"
                            @click="deleteEmployee(employee.id)">Delete</button>
                    </td>
                </tr>
            </template>
        </tbody>
    </table>
</template>
<script>
    import {
        onMounted,
        reactive,
        ref
    } from 'vue';
    import axios from 'axios';



    export default {

        setup() {
            const employees = ref([]);
            const errors = ref([]);
            const employee_id = ref([]);

            const form = reactive({
                name: '',
                email: '',
                phone: ''
            });

            const getEmployee = async () => {
                let res = await axios.get('api/employees');
                console.log(res);
                employees.value = res.data;
            }

            const deleteEmployee = async (id) => {
                try {
                    await axios.delete('api/employees/' + id)
                    getEmployee();
                } catch (error) {

                }
            }

            const storeEmployee = async () => {
                // console.log(form);
                try {
                    let res = await axios.post('api/employees', form)
                    // console.log(res);
                    getEmployee();
                    formReset();

                    $("#exampleModal").modal('hide');
                } catch (e) {
                    if (e.response.status == 422) {
                        var data = [];
                        for (const key in e.response.data.errors) {
                            data.push(e.response.data.errors[key][0]);
                        }
                        errors.value = data;
                    }
                }
            }

            const updateEmployee = async () => {
                // console.log(form);
                try {
                    let res = await axios.put('api/employees/' +employee_id.value, form)
                    getEmployee();
                    formReset();
                    $("#exampleModal").modal('hide');
                } catch (e) {
                    if (e.response.status == 422) {
                        var data = [];
                        for (const key in e.response.data.errors) {
                            data.push(e.response.data.errors[key][0]);
                        }
                        errors.value = data;
                    }
                }
            }

            const editEmployee = (employee) => {
                employee_id.value = employee.id;
                form.name =  employee.name;
                form.email = employee.email
                form.phone =  employee.phone;
            }

            const formReset = () => {
                employee_id.value = '',
                form.name = '';
                form.email = '';
                form.phone = '';
            }
            onMounted(getEmployee())
            // console.log(employees);
            return {
                employees,
                form,
                errors,
                storeEmployee,
                deleteEmployee,
                formReset,
                editEmployee,
                employee_id,
                updateEmployee,
                
            }
        }
    }
</script>
