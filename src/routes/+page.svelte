<script>
  import { Client } from "@dxos/client";
  import { Expando } from "@dxos/client/echo";

  // create a client
  const client = new Client();

  let tasks = [];
  let space;
  let identity;
  let text = "";

  const main = async () => {
    await client.initialize();

    identity = client.halo.identity.get();

    const spaces = client.spaces.get();

    if (spaces.length > 0) {
      const key = spaces[0].key;

      space = client.getSpace(key);
    } else {
      space = await client.createSpace({ name: "todos" });
    }
  };

  main();

  $: if (space) {
    space.db.query({ type: "task" }).subscribe((a) => {
      tasks = a;
    });
  }

  const handleSubmit = () => {
    // Create object
    const object = new Expando({ type: "task", title: text });

    // Adds object
    space.db.add(object);

    // Resets text input
    text = "";
  };
</script>

<h1>Todos</h1>

<form on:submit|preventDefault={() => handleSubmit()}>
  <input bind:value={text} />
  <input type="submit" value="add task" />
</form>

{#if tasks.objects}
  <ul>
    {#each tasks.objects as task}
      <li>Title: {task.title}</li>
    {/each}
  </ul>
{/if}
