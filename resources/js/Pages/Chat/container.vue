// Container

<template>
    <app-layout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                <chat-room-seletion
                    v-if="currentRoom.id"
                    :rooms="chatRooms"
                    :currentRoom="currentRoom"
                    v-on:roomchanged="setRoom($event)"
                />
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <messages-container :messages="messages" />
                    <input-message
                        :room="currentRoom"
                        v-on:messagesent="getMessages()"
                    />
                </div>
            </div>
        </div>
    </app-layout>
</template>

<script>
import AppLayout from "@/Layouts/AppLayout";
import MessagesContainer from "./messagesContainer.vue";
import InputMessage from "./inputMessage.vue";
import ChatRoomSeletion from "./chatRoomSeletion.vue";

export default {
    components: {
        AppLayout,
        MessagesContainer,
        InputMessage,
        ChatRoomSeletion,
    },
    data: function () {
        return {
            chatRooms: [],
            currentRoom: [],
            messages: [],
        };
    },
    watch: {
        // room and oldroom
        currentRoom(val, oldVal) {
            if (oldVal.id) {
                this.disconnect(oldVal);
            }
            this.connect();
        },
    },
    methods: {
        // user pusher websockets
        connect() {
            if (this.currentRoom.id) {
                let vm = this;
                this.getMessages();
                window.Echo.private("chat." + this.currentRoom.id).listen(
                    ".message.new",
                    (e) => {
                        vm.getMessages();
                    }
                );
            }
        },
        disconnect(room) {
            window.Echo.leave("chat." + room.id);
        },
        //all rooms
        getRooms() {
            axios
                .get("/chat/rooms")
                .then((response) => {
                    // handle success
                    this.chatRooms = response.data; // all
                    this.setRoom(response.data[0]); //by default set room = first room available
                })
                .catch((error) => {
                    // handle error
                    console.log(error);
                });
        },
        setRoom(room) {
            this.currentRoom = room;
            //this.getMessages();
        },
        //get  messages for our current room
        getMessages() {
            axios
                .get("/chat/room/" + this.currentRoom.id + "/messages")
                .then((response) => {
                    this.messages = response.data;
                })
                .catch((error) => {
                    console.log(error);
                });
        },
    },
    created() {
        this.getRooms();
    },
};
</script>
