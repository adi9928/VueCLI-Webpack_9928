<template>
        <v-main class="list">
            <h3 class="text-h3 font-weight-medium mb-5">
                To Do List
            </h3>
            <div class="d-flex align-start mr-3">
            <v-card>
                <v-card-title>
                    <v-text-field
                        v-model="search"
                        append-icon="mdi-magnify"
                        label="Search"
                        single-line
                        hide-details
                    ></v-text-field>
                    <v-spacer></v-spacer>
                    <v-btn color="success" dark @click="dialog = true">
                        Tambah
                    </v-btn>
                </v-card-title>
                <v-data-table :headers="headers" :items="todos" :search="search">
                    <template v-slot:[`item.priority`]="{ item }">
                        <v-chip
                            v-if="item.priority == 'Penting'"
                            class="ma-2"
                            color="red"
                            label
                            outlined
                        >
                            {{item.priority}}
                        </v-chip>
                        <v-chip
                            v-else-if="item.priority == 'Tidak penting'"
                            class="ma-2"
                            color="green"
                            label
                            outlined
                        >
                            {{item.priority}}
                        </v-chip>
                        <v-chip
                            v-else
                            class="ma-2"
                            color="blue"
                            label
                            outlined
                        >
                            {{item.priority}}
                        </v-chip>
                    </template>
                    <template v-slot:[`item.actions`]="{ item }">
                        <v-icon 
                        class="ma-4"
                        @click="detail(item)"
                        color="blue">
                            mdi-file
                        </v-icon>
                    </template>
                    <template v-slot:[`item.checkbox`]="{ item }">
                        <input type="checkbox" v-model="item.checked" @change="kenaCheck(item)">
                    </template>
                </v-data-table>
            </v-card>
            <v-card flat card-btn-padding class="ma-3" v-show="cardDetail" width="600px">
                <v-card-title>
                     <h1 small class="ma-2">
                        {{tempTodo.task}}
                    </h1>
                    <v-spacer></v-spacer>
                    <v-chip
                        v-if="tempTodo.priority == 'Penting'"
                        class="ma-2"
                        color="red"
                        label
                        text-color="white"
                    >
                        {{tempTodo.priority}}
                    </v-chip>
                    <v-chip
                        v-else-if="tempTodo.priority == 'Tidak penting'"
                        class="ma-2"
                        color="green"
                        label
                        text-color="white"
                    >
                        {{tempTodo.priority}}
                    </v-chip>
                    <v-chip
                        v-else
                        class="ma-2"
                        color="blue"
                        label
                        text-color="white"
                    >
                        {{tempTodo.priority}}
                    </v-chip>
                </v-card-title>
                <h3 small class="ma-4" >
                    Note :
                    <br>
                    {{tempTodo.note}}
                </h3>
                <v-icon 
                 class="ma-4"
                 @click="editItem()"
                 color="blue">
                    mdi-pencil
                </v-icon>
                <v-icon 
                class="ma-2"
                @click="deleteItem()"
                color="red">
                    mdi-delete
                </v-icon>
            </v-card>
            
            <v-dialog v-model="dialog" persistent max-width="600px">
                <v-card>
                    <v-card-title>
                    <span class="headline">
                        Form Todo
                        </span>
                    </v-card-title>
                    <v-card-text>
                        <v-container>
                            <v-text-field
                                v-model="formTodo.task"
                                label="Task"
                                required
                            ></v-text-field>
                            <v-select
                                v-model="formTodo.priority"
                                :items="['Penting', 'Biasa', 'Tidak penting']"
                                label="Priority"
                                required
                            ></v-select>
                            <v-textarea
                                v-model="formTodo.note"
                                label="Note"
                                required
                            ></v-textarea>
                        </v-container>
                    </v-card-text>
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="blue darken-1" text @click="cancel">
                            Cancel
                        </v-btn>
                        <v-btn color="blue darken-1" text @click="save" v-if="!isEdit">
                            Save
                        </v-btn>
                        <v-btn color="blue darken-1" text @click="edit" v-else>
                            Save
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>

            <v-dialog v-model="dialog2" persistent max-width="600px">
                <v-card>
                    <v-card-title>
                    <span class="headline">
                        Yakin ingin hapus ?
                        </span>
                    </v-card-title>
                    
                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="blue darken-1" text @click="cancel">
                            Tidak
                        </v-btn>
                        <v-btn color="blue darken-1" text @click="deletes">
                            Ya
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
            </div>
            <v-card width="1600px" v-show="displayTugas">
                <v-card-title>
                    <span class="headline">
                        Delete Multiple :
                    </span>
                    <v-spacer></v-spacer>
                    <v-btn color="red" dark @click="deleteAll(0)">
                        Delete All
                    </v-btn>
                </v-card-title>
                <div v-for="todo in todos" :key="todo.task">
                    <h5 class="ma-2 ml-4" v-if="todo.checked">
                        {{todo.task}}
                    </h5>
                </div>
            </v-card>
        </v-main>
    
</template>
<script>
    export default {
        name: "List",
        data() {
            return {
                search: null,
                dialog: false,
                dialog2: false,
                cardDetail: false,
                isEdit: false,
                index: null,
                checked:false,
                displayTugas: false,
                tempItem:null,
                headers: [
                    {
                        text: "Task",
                        align: "start",
                        sortable: true,
                        value: "task",
                    },
                    { text: "Priority", value: "priority" },
                    { text: "Actions", value: "actions" },
                    { text: "", value: "checkbox" },
                ],
                todos: [
                    {
                        task: "bernafas",
                        priority: "Penting",
                        note: "huffttt",
                        checked: false,
                    },
                    {
                        task: "nongkrong",
                        priority: "Tidak penting",
                        note: "bersama tman2",
                        checked:false,
                    },
                    {
                        task: "masak",
                        priority: "Biasa",
                        note: "masak air 500ml",
                        checked:false,
                    },
                ],
                deleteList:[

                ],
                formTodo: {
                    task: null,
                    priority: null,
                    note: null,
                    checked: false,
                },
                tempTodo:{
                    task: "tes",
                    priority:"Penting",
                    note: "tes",
                },
            };
        },
        methods: {
            save() {
                this.todos.push(this.formTodo);
                this.resetForm();
                this.dialog = false;
            },
            cancel() {
                this.resetForm();
                this.dialog = false;
                this.dialog2 = false;
            },
            resetForm() {
                this.formTodo = {
                task: null,
                priority: null,
                note: null,
                };
            },
            edit(){
                this.index = this.todos.indexOf(this.tempItem);
                this.todos[this.index].task = this.formTodo.task;
                this.todos[this.index].priority = this.formTodo.priority;
                this.todos[this.index].note = this.formTodo.note;
                this.resetForm();
                this.dialog = false;
                this.isEdit = false;
                this.detail(this.todos[this.index]);
            },
            editItem(){
                this.dialog = true;
                this.isEdit = true;
                this.formTodo.task = this.tempTodo.task;
                this.formTodo.priority = this.tempTodo.priority;
                this.formTodo.note = this.tempTodo.note;
            },
            deletes(){
                this.todos.splice(this.index,1);
                this.dialog2 = false;
                this.cardDetail=false;
                this.tempTodo.task = null;
                this.tempTodo.priority = null;
                this.tempTodo.note = null;
            },
            deleteItem(){
                this.index = this.todos.indexOf(this.tempItem);
                this.dialog2 = true;
            },
            detail(item){
                this.cardDetail = true;
                
                this.tempTodo.task = item.task;
                this.tempTodo.priority = item.priority;
                this.tempTodo.note = item.note;
                this.tempItem=item;
            },
            kenaCheck(item){
                this.displayTugas=true;
                this.tempItem=item;
            },
             deleteAll(index){
                for (index = 0; index < this.todos.length; index++){
                    if(this.todos[index].checked){
                        if(this.todos[index]==this.tempItem)
                            this.cardDetail=false;
                        this.todos.splice(index,1);
                        index=-1;
                        
                    }
                }
                
            },
        },
    };
</script>
