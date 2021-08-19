<script>
  import { initializeApp, getApps, getApp } from "firebase/app";
  import {
    getFirestore,
    collection,
    onSnapshot,
    doc,
    addDoc,
    updateDoc,
    deleteDoc,
  } from "firebase/firestore";
  import { firebaseConfig } from "$lib/firebaseConfig";
  import { browser } from "$app/env";

  const firebaseApp =
    browser &&
    (getApps().length === 0 ? initializeApp(firebaseConfig) : getApp());

  const db = browser && getFirestore();

  const colRef = browser && collection(db, "todos");

  let todos = [];

  const unsubscribe =
    browser &&
    onSnapshot(colRef, (querySnapshot) => {
      let fbTodos = [];
      querySnapshot.forEach((doc) => {
        let todo = { ...doc.data(), id: doc.id };
        fbTodos = [todo, ...fbTodos];
      });
      todos = fbTodos;
    });


  let task = "";
  let error = "";

  const addTodo = async () => {
    if (task !== "") {
      const docRef = await addDoc(collection(db, "todos"), {
        task: task,
        isComplete: false,
        createdAt: new Date(),
      });
      error = "";
    } else {
      error = "Task is empty";
    }
    task = "";
  };

  const markTodoAsComplete = async (item) => {
    await updateDoc(doc(db, "todos", item.id), {
      isComplete: !item.isComplete,
    });
  };

  const deleteTodo = async (id) => {
    await deleteDoc(doc(db, "todos", id));
  };

  const keyIsPressed = (event) => {
    if (event.key === "Enter") {
      addTodo();
    }
  };
</script>

<input type="text" placeholder="Add a task" bind:value={task} />
<button on:click={addTodo}>Add</button>

<ol>
  {#each todos as item}
    <li class:complete={item.isComplete}>
      <span>
        {item.task}
      </span>
      <span>
        <button on:click={() => markTodoAsComplete(item)}>✔</button>
        <button on:click={() => deleteTodo(item.id)}>✘</button>
      </span>
    </li>
  {:else}
    <p>No todos found</p>
  {/each}
  <p class="error">{error}</p>
</ol>

<svelte:window on:keydown={keyIsPressed} />

<style>
  .complete {
    text-decoration: line-through;
  }
  .error {
    color: red;
  }
</style>
