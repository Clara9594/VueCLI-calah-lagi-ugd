<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model ="search"
                    append-icon ="mdi-magnify"
                    label = "Search"
                    single-line
                    hide-details
                ></v-text-field>
                <v-spacer></v-spacer>

                <v-form>
                    <v-autocomplete
                        ref="country"
                        v-model="selected" :items="['Penting', 'Tidak Penting']"
                        label="Urutkan"
                    ></v-autocomplete>
                </v-form>

                <v-btn color = "success" dark @click = "dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>

            <v-data-table :headers = "headers" :items = "todos" :search = "search"
            :single = "search"
            :single-expand = "singleExpand"
            :expanded.sync = "expanded"
            item-key = "note"
            show-expand
            class = "elevation-1">

                <template v-slot:[`item.priority`]="{ item }">
                    <td>
                        <v-chip v-if="item.priority == 'Penting'" color="red" outlined label>
                            {{ item.priority }}
                        </v-chip>

                        <v-chip v-else-if="item.priority == 'Biasa'" color="green" outlined label>
                            {{ item.priority }}
                        </v-chip>
                        
                        <v-chip v-else color="blue" outlined label>
                            {{ item.priority }}
                        </v-chip>
                    </td>
                </template>
                
                
                <template v-slot:expanded-item="{ headers, item }">
                    <td :colspan="headers.length">
                        <br>
                        <h2 class="text-left"> Note : </h2>
                        <br>
                        <p class="text-left">{{ item.note }}</p>
                        <br>
                        <br>
                    </td>
                </template>


                <template v-slot:[`item.actions`]= "{ item }">
                    <v-btn small class = "mr-2" @click = "editItem(item)">
                        edit
                    </v-btn>
                    <v-btn small @click = "deleteItem(item)">
                        delete
                    </v-btn>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog v-model="modalDelete" persistent max-width="400px">
            <v-card>
                <v-card-title > 
                    <span class="headline font-weight-bold">Yakin ingin menghapus?</span>
                </v-card-title>

                <v-card-actions>
                    <v-spacer></v-spacer>

                    <v-btn color="green darken-1" text @click="Tidak">
                        Tidak
                    </v-btn>

                    <v-btn color="red darken-1" text @click="Ya">
                        Ya
                    </v-btn>                    
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="modalEdit" persistent max-width="400px">
            <v-card>
                <v-card-title>
                    <span class="headline font-weight-medium">{{ dt.task }}</span>
                </v-card-title>
                <v-card-text class="text-left">                
                        <v-chip v-if= "dt.priority == 'Penting'" color="red" outlined>
                            {{ dt.priority }}
                        </v-chip>

                        <v-chip v-else-if="dt.priority == 'Biasa'" color="green" outlined>
                            {{ dt.priority }}
                        </v-chip>

                        <v-chip v-else color="blue" outlined>
                            {{ dt.priority }}
                        </v-chip>                                        
                </v-card-text>
                <v-card-text class="text-left">
                    {{dt.note}}
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Close
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model = "dialog" persistent max-width = "600px">
            <v-card>
                <v-card-title>
                    <span class = "headline">Form Todo</span>
                </v-card-title>

                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model = "formTodo.task"
                            label = "Task"
                            required
                        ></v-text-field>

                        <v-select
                            v-model = "formTodo.priority"
                            :items = "['Penting', 'Biasa', 'Tidak penting']"
                            label = "Priority"
                            required
                        ></v-select>

                    <v-textarea
                            v-model = "formTodo.note"
                            label = "Note"
                            required
                        ></v-textarea> 
                    </v-container>
                </v-card-text>

                <v-card-actions>

                    <v-spacer></v-spacer>

                    <v-btn color="blue darken-1" text @click="Tidak">
                        Cancel
                    </v-btn>

                    <v-btn v-if="editCheck == true" color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                    
                    <v-btn v-else color="blue darken-1" text @click="edit(formTodo)">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-main>
</template>

<script>
export default {
name : "List",
data() {
    return {
    search : null,
    dialog : false,
    modalDelete: false,
    modalEdit: false,
    itemTemp: null,
    editCheck: true,
    headers : [
        {
            text : "Task",
            align : "start",
            sortable : true,
            value : "task",
        },
        { text : "Priority", value : "priority" },
        { text : "Actions", value : "actions" },
    ],

    todos : [
        {
            task : "bernafas",
            priority : "Penting",
            note : "huffttt",
        },
        {
            task : "nongkrong",
            priority : "Tidak penting",
            note : "bersama tman2",
        },
        {
            task : "masak",
            priority : "Biasa",
            note : "masak air 500ml",
        },
    ],

    formTodo : {
        task : null,
        priority : null,
        note : null,
    },

    dt: {
        task: null,
        priority: null,
        note: null,
    },
    };
},

methods: {
    //ini fungsi tambah
    save() {
        this.todos.push(this.formTodo);
        this.resetForm();
        this.dialog = false;
    },
    Tidak() {
        this.resetForm();
        this.dialog = false;
        this.modalDelete = false;
        this.modalEdit = false;
        this.editCheck = true;
        this.itemTemp = null;
    },
    resetForm() {
        this.formTodo = {
            task: null,
            priority: null,
            note: null,
        };
    },

    //ini fungsi edit
    editItem(item) {
        this.editCheck = false;
        this.dialog = true;
        this.formTodo = {
            task : item.task,
            priority : item.priority,
            note : item.note,
        };
        this.itemTemp = item;
    },
    edit(formTodo) {
        this.itemTemp.task = formTodo.task;
        this.itemTemp.priority = formTodo.priority;
        this.itemTemp.note = formTodo.note;
        this.Tidak();            
    },

    //ini fungsi hapus
    deleteItem(item) {
        this.modalDelete = true;
        this.itemTemp = item;
    },
    Ya() {
        this.todos.splice(this.todos.indexOf(this.itemTemp), 1);
        this.dialogdel = false;
        this.Tidak();
    },
    detailItem(item) {
        this.modalEdit= true;
        this.dt = {
            task : item.task,
            priority : item.priority,
            note : item.note,
        }
    },
},
};
</script>
