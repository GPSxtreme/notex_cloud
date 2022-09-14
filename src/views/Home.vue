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
    <div class="flex items-center justify-between">
      <h1 class="logo"><span class="logo1">Notex </span>cloud☁</h1>
      <button @click="logout" class="logout-btn">Logout</button>
    </div>
    <hr class="top-line-divider" />
    <form @submit.prevent="handleCreateNote" class="mb-8">
      <div class="noteInputBox">
        <label class="block mb-4">
          <label class="block mb-4 mt-4"
            ><span class="block text-base uppercase mb-2">Title</span></label
          >
          <input
            type="text"
            v-model="newNote.title"
            placeholder="Title"
            class="block w-full p-2 border border-gray-300 rounded-md text-slate-800"
          />
          <label class="block mb-4 mt-4"
            ><span class="block text-base uppercase mb-2">Content</span></label
          >
          <textarea
            type="text"
            v-model="newNote.content"
            placeholder="Write your note here..."
            class="block w-full p-2 border border-gray-300 rounded-md text-slate-800"
          ></textarea>
        </label>
        <input type="submit" value="create note" class="create-note-btn" />
      </div>
      <hr class="top-line-divider" />
    </form>
    <div v-if="!loadingNotes">
      <div
        v-for="note in noteResults.notes"
        :key="note.id"
        class="relative shadow-xl bg-gray-500/40 text-white rounded-lg p-6 mb-6 noteBox"
      >
        <h3 class="font-bold text-2xl mb-3 noteTitle">{{ note.title }}</h3>
        <button
          class="absolute right-4 top-20 delete-btn"
          @click="() => deleteNote({ id: note.id })"
        >
          Delete
        </button>
        <p class="text-lg mb-3" v-html="convertToHTML(note.content)"></p>
        <div class="text-sm text-green-500 italic">
          {{ convertToData(note.created) }}
        </div>
      </div>
    </div>
  </main>
</template>
<style>
@import url("https://fonts.googleapis.com/css2?family=Inter&family=Pacifico&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Inconsolata:wght@500&display=swap");
main {
  background-color: blue;
  min-height: 100vh;
  color: white;
  background-color: #c8eed9;
  font-family: "Inconsolata", monospace;
}

.logo {
  margin: 10px auto;
  text-align: center;
  font-family: "Bebas Neue", cursive;
  font-size: 2rem;
  letter-spacing: 3px;
  background-color: #fff1bd;
  width: 50%;
  color: #2eb2ff;
  padding: 7px 10px;
  border: 2px dashed rgb(35, 37, 85);
  border-radius: 8px;
  box-shadow: 6.7px 6.7px 5.3px rgba(0, 0, 0, 0.041),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.059),
    100px 100px 80px rgba(0, 0, 0, 0.12);
}
.logo1 {
  font-family: "Pacifico", cursive;
  letter-spacing: none;
  color: #146356;
}
.logout-btn {
  background-color: white;
  color: red;
  font-family: "Bebas Neue", cursive;
  padding: 7px 10px;
  font-size: 1.5rem;
  border-radius: 8px;
  border: 2px dashed red;
}
.create-note-btn {
  background-color: white;
  color: green;
  font-family: "Bebas Neue", cursive;
  padding: 7px 10px;
  font-size: 1rem;
  border-radius: 8px;
  border: 2px dashed green;
}
.logout-btn:hover {
  color: white;
  background: rgba(233, 9, 9, 0.489);
  box-shadow: 2.8px 2.8px 2.2px rgba(0, 0, 0, 0.052),
    6.7px 6.7px 5.3px rgba(0, 0, 0, 0.068),
    12.5px 12.5px 10px rgba(0, 0, 0, 0.081),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.099),
    41.8px 41.8px 33.4px rgba(0, 0, 0, 0.128),
    100px 100px 80px rgba(0, 0, 0, 0.2);
}
.top-line-divider {
  height: 1px;
  margin: 0 auto;
  margin-top: 36px;
  margin-bottom: 36px;
  width: 90%;
  border: 0;
  background-color: #91886a;
  filter: blur(0.8px);
}
.noteInputBox,
.noteBox {
  box-shadow: 2.8px 2.8px 2.2px rgba(0, 0, 0, 0.02),
    6.7px 6.7px 5.3px rgba(0, 0, 0, 0.028),
    12.5px 12.5px 10px rgba(0, 0, 0, 0.035),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.042),
    41.8px 41.8px 33.4px rgba(0, 0, 0, 0.05),
    100px 100px 80px rgba(0, 0, 0, 0.07);
  width: 90%;
  margin: 0 auto;
  background-color: #fff1bd;
  font-size: 1.2rem;
  color: #146356;
  padding: 20px;
  overflow: scroll;
}
.noteBox {
  margin-top: 20px;
}
.delete-btn {
  background-color: white;
  color: red;
  font-family: "Bebas Neue", cursive;
  padding: 7px 10px;
  font-size: 1rem;
  border-radius: 8px;
  border: 2px dashed red;
}
.delete-btn:hover {
  color: white;
  background: rgba(233, 9, 9, 0.489);
  box-shadow: 2.8px 2.8px 2.2px rgba(0, 0, 0, 0.052),
    6.7px 6.7px 5.3px rgba(0, 0, 0, 0.068),
    12.5px 12.5px 10px rgba(0, 0, 0, 0.081),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.099),
    41.8px 41.8px 33.4px rgba(0, 0, 0, 0.128),
    100px 100px 80px rgba(0, 0, 0, 0.2);
}
.create-note-btn:hover {
  color: white;
  background: rgba(46, 233, 9, 0.489);
  box-shadow: 2.8px 2.8px 2.2px rgba(0, 0, 0, 0.052),
    6.7px 6.7px 5.3px rgba(0, 0, 0, 0.068),
    12.5px 12.5px 10px rgba(0, 0, 0, 0.081),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.099),
    41.8px 41.8px 33.4px rgba(0, 0, 0, 0.128),
    100px 100px 80px rgba(0, 0, 0, 0.2);
}
@media only screen and (max-width: 430px) {
  .logo {
    font-size: 120%;
  }
  .logout-btn,
  .create-note-btn,
  .delete-btn {
    font-size: 80%;
  }
}
</style>
