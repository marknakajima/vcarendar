<template>
    <div>
        <v-navigation-drawer
            v-model="drawerSwitch"
            :permanent="$vuetify.breakpoint.lgAndUp"
            :absolute="$vuetify.breakpoint.mdAndDown"
            floating
            right
        >
            <v-list>
                <v-list-item-group>
                    <v-list-item
                        link
                        @click="eventsByName('', '')"
                        class="mt-n2"
                        :class="
                            getSelectedPerson.name === '' ||
                            !getSelectedPerson.name
                                ? 'blue'
                                : 'white'
                        "
                    >
                        <v-list-item-title
                            :class="
                                getSelectedPerson.name === '' ||
                                !getSelectedPerson.name
                                    ? 'white--text'
                                    : ''
                            "
                            ><span class="mr-2">All</span></v-list-item-title
                        >
                    </v-list-item>
                </v-list-item-group>

                <v-list-group :value="true" dense>
                    <template v-slot:activator>
                        <v-list-item-title
                            ><span class="mr-3">Caregivers</span>
                            <span class="caption grey--text">{{
                                namesCaregivers.length
                            }}</span></v-list-item-title
                        >
                    </template>
                    <v-container>
                        <v-form ref="newCaregiverForm" v-model="valid">
                            <v-row no-gutters>
                                <v-col cols="8" sm="8">
                                    <v-text-field
                                        placeholder="Add Caregiver"
                                        v-model="newCaregiver"
                                        v-on:input="
                                            isDuplicateName = checkDuplicateName(
                                                newCaregiver,
                                                'namesCaregivers'
                                            )
                                        "
                                        :rules="rulesName"
                                    ></v-text-field>
                                </v-col>
                                <v-col cols="4" sm="4">
                                    <v-btn
                                        class="mt-4 ml-4"
                                        small
                                        color="blue"
                                        depressed
                                        @click="
                                            addNewPerson(
                                                newCaregiver,
                                                'newCaregiver'
                                            )
                                        "
                                        :disabled="
                                            !valid ||
                                            isDuplicateName ||
                                            newCaregiver === '' ||
                                            newCaregiver === undefined
                                        "
                                        ><span class="white--text"
                                            >Add</span
                                        ></v-btn
                                    >
                                </v-col>
                            </v-row>
                        </v-form>
                    </v-container>
                    <v-divider></v-divider>
                    <v-virtual-scroll
                        :items="namesCaregivers"
                        :item-height="40"
                        height="220"
                    >
                        <template v-slot:default="{ item }">
                            <v-list-item
                                link
                                dense
                                @click="eventsByName(item, 'caregiver')"
                                :class="
                                    item === getSelectedPerson.name
                                        ? 'blue'
                                        : ''
                                "
                            >
                                <v-list-item-title
                                    :class="
                                        item === getSelectedPerson.name
                                            ? 'white--text'
                                            : ''
                                    "
                                    >{{ item }}
                                </v-list-item-title>
                            </v-list-item>
                        </template>
                    </v-virtual-scroll>
                </v-list-group>

                <v-list-group :value="true">
                    <template v-slot:activator>
                        <v-list-item-title
                            ><span class="mr-3">Clients</span>
                            <span class="caption grey--text">{{
                                namesClients.length
                            }}</span></v-list-item-title
                        >
                    </template>

                    <v-container>
                        <v-form ref="newClientForm" v-model="valid">
                            <v-row no-gutters>
                                <v-col cols="8" sm="8">
                                    <v-text-field
                                        placeholder="Add Client"
                                        v-model="newClient"
                                        v-on:input="
                                            isDuplicateName = checkDuplicateName(
                                                newClient,
                                                'namesClients'
                                            )
                                        "
                                        :rules="rulesName"
                                    ></v-text-field>
                                </v-col>
                                <v-col cols="4" sm="4">
                                    <v-btn
                                        class="mt-4 ml-4"
                                        small
                                        color="blue"
                                        depressed
                                        @click="
                                            addNewPerson(newClient, 'newClient')
                                        "
                                        :disabled="
                                            !valid ||
                                            isDuplicateName ||
                                            newClient === '' ||
                                            n4wClient === undefined
                                        "
                                        ><span class="white--text"
                                            >Add</span
                                        ></v-btn
                                    >
                                </v-col>
                            </v-row>
                        </v-form>
                    </v-container>
                    <v-divider></v-divider>

                    <v-virtual-scroll
                        :items="namesClients"
                        :item-height="40"
                        height="220"
                    >
                        <template v-slot:default="{ item }">
                            <v-list-item
                                link
                                dense
                                @click="eventsByName(item, 'client')"
                                :class="
                                    item === getSelectedPerson.name
                                        ? 'blue'
                                        : ''
                                "
                            >
                                <v-list-item-title
                                    :class="
                                        item === getSelectedPerson.name
                                            ? 'white--text'
                                            : ''
                                    "
                                    >{{ item }}
                                </v-list-item-title>
                            </v-list-item>
                        </template>
                    </v-virtual-scroll>
                </v-list-group>
            </v-list>
        </v-navigation-drawer>
    </div>
</template>

<script>
import axios from "axios";
import { mapGetters, mapActions, mapMutations } from "vuex";
export default {
    name: "CalendarSideBar",
    props: {
        focus: {
            type: String,
            default: "",
        },
    },
    data() {
        return {
            valid: false,
            loading: false,
            items: [],
            newCaregiver: "",
            newClient: "",
            isDuplicateName: false,
            rulesName: [
                (value) =>
                    !value ||
                    /^[a-zA-Z&-]+\s{1}[a-zA-Z-]+$/.test(value) ||
                    "Names must be 2 words separated by space",
                (value) => !!value || "Required",
            ],
        };
    },
    watch: {},
    computed: {
        ...mapGetters(["drawerStatus"]),
        ...mapGetters("storeCalendar", [
            "getSelectedPerson",
            "getNamesCaregivers",
            "getTempCaregiver",
            "getNamesClients",
            "getTempClient",
        ]),
        drawerSwitch: {
            get: function () {
                return this.drawerStatus;
            },
            set: function () {},
        },
        namesCaregivers: {
            get: function () {
                return [...this.getNamesCaregivers, this.getTempCaregiver]
                    .filter((item) => item !== "")
                    .map((event) => event)
                    .sort((a, b) => {
                        return a.split(" ")[1].localeCompare(b.split(" ")[1]);
                    });
            },
        },
        namesClients: {
            get: function () {
                return [...this.getNamesClients, this.getTempClient]
                    .filter((item) => item !== "")
                    .map((event) => event)
                    .sort((a, b) => {
                        return a.split(" ")[1].localeCompare(b.split(" ")[1]);
                    });
            },
        },
    },
    methods: {
        ...mapActions(["updateSnackMessage"]),
        ...mapActions("storeCalendar", ["updateSelectedPerson"]),
        ...mapMutations(["drawerChange"]),
        ...mapMutations("storeCalendar", ["ADD_NEW_TEMPORARY_PERSON"]),
        async eventsByName(name, type) {
            await this.updateSelectedPerson({ name, type });
            this.$emit("selectedPerson");
            this.drawerChange();
        },
        checkDuplicateName(name, type) {
            return this[type].includes(name);
        },
        addNewPerson(name, type) {
            // Store new temporary person in Vuex
            let nameType =
                type === "newCaregiver" ? "caregiverNames" : "clientNames";
            this.ADD_NEW_TEMPORARY_PERSON({ name, nameType });

            // Select this new person in sidebar
            let personType = type === "newCaregiver" ? "caregiver" : "client";
            this.eventsByName(name, personType);

            // Clear the text field.
            let formType =
                type === "newCaregiver" ? "newCaregiverForm" : "newClientForm";
            this.$refs[formType].reset();

            this.newCaregiver = "";
            this.newClient = "";
            this.isDuplicateName = false;

            this.updateSnackMessage(`Added new temporary person : ${name}`);
        },
        clearItems() {
            this.items = [];
        },
    },
};
</script>

<style scoped>
</style>