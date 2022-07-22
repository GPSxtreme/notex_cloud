<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import { useSignOut, useUserId } from "@nhost/vue";
import { useMutation, useQuery } from "@vue/apollo-composable";
import { gql } from "@apollo/client/core";

const router = useRouter();
const { signOut } = useSignOut();
const userId = useUserId();

const newNote = ref({
  title: "",
  content: "",
});
const logout = () => {
  signOut();
  router.push("/login");
};

const {
  loading: loadingNotes,
  result: noteResults,
  refetch: refetchResults,
} = useQuery(
  gql`
    query ($userId: String!) {
      notes(order_by: { created: desc }, where: { user_id: { _eq: $userId } }) {
        id
        title
        content
        created
      }
    }
  `,
  {
    userId: userId.value,
  }
);

const { mutate: createNote, onDone: createNoteDone } = useMutation(gql`
  mutation ($userId: String!, $title: String!, $content: String!) {
    insert_notes_one(
      object: { content: $content, title: $title, user_id: $userId }
    ) {
      id
    }
  }
`);

const handleCreateNote = () => {
  if (!newNote.value.title || !newNote.value.content) {
    return alert("Please fill in all fields");
  }
  createNote({
    userId: userId.value,
    title: newNote.value.title,
    content: newNote.value.content,
  });
};

createNoteDone(() => {
  refetchResults();
  newNote.value = {
    title: "",
    content: "",
  };
});

const convertToHTML = (content) => {
  return content
    .replace(/\n/g, "<br>")
    .replace(/\*\*(.*?)\*\*/g, "<b>$1</b>")
    .replace(/\*(.*?)\*/g, "<i>$1</i>");
};

const { mutate: deleteNote, onDone: deleteDone } = useMutation(gql`
  mutation ($id: Int!) {
    delete_notes(where: { id: { _eq: $id } }) {
      affected_rows
    }
  }
`);

deleteDone(() => {
  refetchResults();
});

const convertToData = (date) => {
  return new Date(date).toLocaleString();
};
</script>

<template>
  <main>
    <div class="flex items-center justify-between mb-8">
      <h1 class="text-3xl font-bold">Notes with Auth👌</h1>
      <button @click="logout" class="text-red-500 text-sm font-bold">
        Logout
      </button>
    </div>

    <form @submit.prevent="handleCreateNote" class="mb-8">
      <label class="block mb-4">
        <label class="block mb-4 mt-4"
          ><span class="block text-sm uppercase mb-2">Title</span></label
        >
        <input
          type="text"
          v-model="newNote.title"
          placeholder="Title"
          class="block w-full p-2 border border-gray-300 rounded-md text-slate-800"
        />
        <label class="block mb-4 mt-4"
          ><span class="block text-sm uppercase mb-2">Content</span></label
        >
        <textarea
          type="text"
          v-model="newNote.content"
          placeholder="Write your note here..."
          class="block w-full p-2 border border-gray-300 rounded-md text-slate-800"
        ></textarea>
      </label>
      <input
        type="submit"
        value="create note"
        class="text-green-500 hover:underline cursor-pointer mt-4"
      />
    </form>
    <div v-if="!loadingNotes">
      <div
        v-for="note in noteResults.notes"
        :key="note.id"
        class="relative bg-white text-slate-700 rounded-lg p-6 mb-6"
      >
        <button
          class="absolute right-6 top-6 text-red-500 font-bold"
          @click="() => deleteNote({ id: note.id })"
        >
          Delete
        </button>

        <h3 class="font-bold text-2xl mb-3">{{ note.title }}</h3>
        <p
          class="text-lg mb-3 text-gray-500"
          v-html="convertToHTML(note.content)"
        ></p>
        <div class="text-sm text-gray-500 italic">
          {{ convertToData(note.created) }}
        </div>
      </div>
    </div>
  </main>
</template>
