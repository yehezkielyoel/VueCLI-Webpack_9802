<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List Tugas</h3>

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

                <div class="md-select">
                    <v-select
                        v-model="searchp"
                        label="Priority"
                        :items="['All Priority','Penting', 'Biasa', 'Tidak penting']"
                        outlined
                    ></v-select>
                </div>

                <v-spacer></v-spacer>

                <v-btn color="success" dark @click="dialog = true">
                    tambah
                </v-btn>
            </v-card-title>

            <v-data-table 
                :headers="headers" 
                :items="filteredItems" 
                :search="search"
                :expanded.sync="expanded"
                item-key="name"
                show-expand
                class="elevation-1">

                <template v-slot:[`item.priority`]="{ item }">
                    <td>
                        <v-card v-if="item.priority == 'Penting'" style="border-color: lightcoral; color: lightcoral; width: fit-content;" outlined>
                            {{ item.priority }}
                        </v-card>
                        <v-card v-else-if="item.priority == 'Biasa'" style="border-color: lightblue; color: lightblue; width: fit-content;" outlined>
                            {{ item.priority }}
                        </v-card>
                        <v-card v-else outlined style="border-color: lightgreen; color: lightgreen; width: fit-content;">
                            {{ item.priority }}
                        </v-card>
                    </td>
                </template>

                <template v-slot:expanded-item="{ headers, item }">
                    <!-- <td>{{ item.task }}</td>
                    <td>{{ item.priority }}</td>
                    <td>{{ item.note }}</td> -->
                    <td :colspan="headers.length">
                        More info about {{ item.note }}
                    </td>
                </template>

                <template v-slot:[`item.actions`]="{ item }">

                    <v-icon small class="pencil mr-2" @click="editItem(item)">{{ icons.mdiPencil }}</v-icon>
                    <v-icon small class="bin mr-2" @click="deleteItem(item)"> {{ icons.mdiDelete }}</v-icon>

                </template>

                <template v-slot:[`item.select`]="{ item }">
                    <v-checkbox multiple :key="item" @click.capture.stop="toggleSelect(item)"/>
                </template>

            </v-data-table>
        </v-card>

        <v-card v-if="selected.length" style="margin-top: 30px; padding: 20px;">
            <v-card-title>
                <h4>Delete Multiple:</h4>
            </v-card-title>
            <v-list-item
                v-for="(item, i) in selected"
                :key="i">
                <v-list-item-content>
                    <v-list-item-title>•  {{item.task}}</v-list-item-title>
                </v-list-item-content>
            </v-list-item>
            <v-btn color="red lighten-2" dark @click="deleteSelected">
                hapus semua
            </v-btn>
        </v-card>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline" 
                        v-if="adding == true" >
                           Form Todo - Add
                    </span>
                    <span class="headline" 
                        v-else>
                           Form Todo - Edit
                    </span>
                </v-card-title>

                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required
                            autofocus
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
                    
                    <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
                    <v-btn v-if="adding == true" 
                        color="blue darken-1" 
                        text 
                        @click="save">
                        Save
                    </v-btn>
                    <v-btn v-else 
                        color="blue darken-1" 
                        text 
                        @click="edit(formTodo)">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="dialogdel" persistent max-width="400px">
            <v-card>
                <v-card-title>
                    <span class="headline">Yakin ingin menghapus?</span>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="green darken-1" text @click="cancel">Tidak</v-btn>
                    <v-btn color="red darken-1" text @click="confirmdelete">Ya</v-btn>
                </v-card-actions>

            </v-card>
        </v-dialog>
    </v-main>
</template>
<script>

import {
    mdiPencil,
    mdiDelete
} from '@mdi/js'

export default {
    name: "List",

    data() {
        return {
            search: null,
            searchp: "All Priority",
            adding: true,
            edititem: null,
            dialog: false,
            dialogdel: false,
            selected: [],
            expanded: [],
            singleExpand: false,
    
            icons: {
                mdiPencil,
                mdiDelete,
            },

            filters: {
                search: '',
                priority: '',
            },

            headers: [
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                
                { 
                    text: "Priority",
                    field: "priority", 
                    value: "priority" 
                },

                // { 
                //     text: "Note", 
                //     field: "note", 
                //     value: "note" 
                // },
                { 
                    text: "Actions", 
                    value: "actions", 
                    sortable: false,
                },
                {
                    text: " ceklist box",
                    value: "select",
                    sortable: false,
                }
            ],

            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                },
                {
                    task: "nongkrong",
                    priority: "Tidak penting",
                    note: "bersama tman2",
                },
                {
                    task: "masak",
                    priority: "Biasa",
                    note: "masak air 500ml",
                },
            ],

            formTodo: {
                task: null,
                priority: null,
                note: null,
            },
            detail: {
                task: null,
                priority: null,
                note: null,
            },
        };
    },

    methods: {
        save() {
            this.todos.push(this.formTodo);
            this.cancel();
        },

        cancel() {
            this.resetForm();
            this.dialog = false;
            this.edititem = null;
            this.adding = true;
            this.dialogdel = false;
        },

        deleteItem(item) {
            this.dialogdel = true;
            this.edititem = item;
        },

        confirmdelete() {
            this.todos.splice(this.todos.indexOf(this.edititem), 1);
            this.dialogdel = false;
        },

        editItem(item) {
            this.adding = false;
            this.formTodo = {
                task: item.task,
                priority: item.priority,
                note: item.note,
            };
            this.dialog = true;
            this.edititem = item;
        },

        edit(formTodo) {
            this.edititem.task = formTodo.task;
            this.edititem.priority = formTodo.priority;
            this.edititem.note = formTodo.note;
            this.cancel();
        },
        
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },

        toggleSelect(item) {
            if(this.selected.includes(item)) {
                this.selected.splice(this.selected.indexOf(item), 1);
            } else {
                this.selected.push(item);
            }
        },

        deleteSelected () {
            for(var i = 0; i < this.selected.length; i++){
                const index = this.todos.indexOf(this.selected[i]);
                this.todos.splice(index, 1);
            }
            this.selected=[];
            toggleSelect(this.todos);
        },
    },

    computed: {
        filteredItems() {
            return this.todos.filter((i) => {
                if(this.searchp !== "All Priority") {
                    return !this.searchp || (i.priority === this.searchp);
                } else {
                    return this.todos;
                }
            })
        }
    }
};
</script>

<style scoped>
.pencil{color: blue !important;}
.bin{color: red !important;}
</style>