<script lang="ts">
  export let users: any;
  export let tasks: any;

  const rankSize = 3;
  const nameSize = 30;
  const taskSize = 6;
  const totalSize = 7;

  let sortKey = "Total";
  let usersSize = Object.entries(users).filter(([name, user]) => Object.keys(user).length > 0).length;
  let totalTasks = 0;
  Object.values(tasks).forEach((tasks: any) => {
    totalTasks += Object.keys(tasks).length;
  });
  let currentHover = "";

  function getScore(user: any, key: string) {
    if(!user[key]) {
      return 0;
    }
    let total = 0;
    Object.values(user[key]).forEach((score: number) => {
      total += score;
    });
    return total;
  }

  function getValue(user: any, key: string = "Total") {
    if (key === "Total") {
      let total = 0;
      Object.values(user).forEach((task: any) => {
        Object.values(task).forEach((score: number) => {
          total += score;
        });
      });
      return total;
    }
    if (tasks[key]) {
      let total = 0;
      Object.keys(tasks[key]).forEach((task: any) => {
        total += getScore(user, task);
      });
      return total;
    }
    return getScore(user, key);
  }

  $: Users = Object.entries(users)
  .filter(([name, user]) => Object.keys(user).length > 0)
  .sort((a, b) => {
    let dif = getValue(b[1], sortKey) - getValue(a[1], sortKey);
    return dif === 0 ? getValue(b[1], "Total") - getValue(a[1], "Total") : dif;
  });

  function RedToGreen(percent: number) {
    let r = Math.floor(255 * (1.25 - percent));
    let g = Math.floor(255 * (0.25 + percent));
    return `rgb(${r},${g},0)`;
  }

  function getRank(user: any, task: string = "Total") {
    let rank = 1;
    for (let i = 0; i < Users.length; i++) {
      if (getValue(Users[i][1], task) > getValue(user, task)) {
        rank++;
      }
    }
    return rank;
  }

  function getMedal(user: any) {
    let rank = getRank(user);
    if (rank <= Math.round(usersSize / 12)) {
      return "gold";
    } else if (rank <= Math.round(usersSize / 4)) {
      return "silver";
    } else if (rank <= Math.round(usersSize / 2)) {
      return "bronze";
    } else {
      return "none";
    }
  }
</script>

<style>
.active {
  background: rgba(255, 255, 255, 0.1);
}
.gold {
  background-color: rgba(255, 215, 0, 0.3);
}
.silver {
  background-color: rgba(192, 192, 192, 0.3);
}
.bronze {
  background-color: rgba(205, 127, 50, 0.3);
}
#scoreboard tr th {
  border-collapse: collapse;
  border-width: 1px;
  border-color: rgb(107, 114, 128, 1);
  border-style: solid;
  padding-left: 0.5rem;
  padding-right: 0.5rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
#scoreboard tr th.score {
  cursor: pointer;
}
#scoreboard tr td {
  border-collapse: collapse;
  border-width: 1px;
  border-color: rgb(107, 114, 128, 1);
  border-style: solid;
  padding-left: 0.5rem;
  padding-right: 0.5rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  white-space: nowrap;
}
#scoreboard tr td.name {
  text-align: left;
  padding-left: 0.75rem;
  text-overflow: ellipsis;
  overflow: hidden;
}
#scoreboard tr td.score {
  font-weight: bold;
}
@media (min-width: 800px) {
  #scoreboard table {
    table-layout: fixed;
  }
}
#details tr th{
  border-collapse: collapse;
  border-width: 1px;
  border-color: rgb(107, 114, 128, 1);
  border-style: solid;
  padding-left: 0.5rem;
  padding-right: 0.5rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  text-align: left;
}
#details tr td{
  border-collapse: collapse;
  border-width: 1px;
  border-color: rgb(107, 114, 128, 1);
  border-style: solid;
  padding-left: 0.25rem;
  padding-right: 0.25rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  text-align: left;
}
#details tr.total {
  background-color: rgba(255, 255, 255, 0.2);
}
#details tr.day {
  background-color: rgba(255, 255, 255, 0.1);
  padding-left: 1rem;
}
#details tr.task{
  padding-left: 1.5rem;
}
#subtask tr th{
  border-collapse: collapse;
  border-width: 1px;
  border-color: rgb(107, 114, 128, 1);
  border-style: solid;
  padding-left: 1rem;
  padding-right: 1rem;
  padding-top: 0.25rem;
  padding-bottom: 0.25rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  text-align: center;
  font-weight: bold;
  font-size: medium;
}
#subtask tr td{
  border-collapse: collapse;
  border-width: 1px;
  border-color: rgb(107, 114, 128, 1);
  border-style: solid;
  padding-left: 1rem;
  padding-right: 1rem;
  padding-top: 0.25rem;
  padding-bottom: 0.25rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  text-align: center;
  font-weight: normal;
  font-size: medium;
}
dialog {
  background-color: #1a202c;
  border-radius: 0.5rem;
  color: white;
  position: fixed;
  border-width: 1px;
  border-color: rgb(107, 114, 128, 1);
  border-style: solid;
  &::backdrop {
    position: fixed;
    top: 0px;
    right: 0px;
    bottom: 0px;
    left: 0px;
    background: rgba(255, 255, 255, 0.1);
  }
}
</style>

<div id="scoreboard">
  <table
    class="w-full border-separate border-spacing-0 rounded-lg border border-gray-500 mx-auto overflow-hidden text-center"
  >
    <thead>
      <tr>
        <th colspan={rankSize}>#</th>
        <th colspan={nameSize}>Name</th>
        {#each Object.entries(tasks) as [day, problems]}
          {#each Object.entries(problems) as [task, name]}
            <th colspan={taskSize} class="score"
              on:click={() => sortKey = task}
            >
              {name}
            </th>
          {/each}
          {#if Object.keys(tasks).length > 1}
            <th colspan={totalSize} class="score"
              on:click={() => sortKey = day}
            >
              {day}
            </th>
          {/if}
        {/each}
        <th colspan={totalSize} class="score"
          on:click={() => sortKey = "Total"}
        >
          Total
        </th>
      </tr>
    </thead>
    <tbody>
      {#each Users as [name, user]}
        <tr
          class:active={currentHover === name}
          on:mouseenter={() => currentHover = name}
          on:mouseleave={() => currentHover = ""}
        >
          <td colspan={rankSize}
            class:gold={getMedal(user) === "gold"}
            class:silver={getMedal(user) === "silver"}
            class:bronze={getMedal(user) === "bronze"}
          >
            {getRank(user)}
          </td>
          <td
            colspan={nameSize} class="name"
            on:click={() => document.getElementById(name).showModal()}
          >
            {name}
          </td>
          {#each Object.entries(tasks) as [day, problems]}
            {#each Object.entries(problems) as [task, _]}
              <td
                colspan={taskSize} class="score"
                style="color: {RedToGreen(getValue(user,task) / 100)}"
                class:active={sortKey === task}
              >{getValue(user,task)}</td>
            {/each}
            {#if Object.keys(tasks).length > 1}
              <td
                colspan={totalSize} class="score"
                style="color: {RedToGreen(getValue(user, day) / (Object.keys(problems).length * 100))}"
                class:active={sortKey === day}
              >{Math.round(getValue(user, day) * 100) / 100}</td>
            {/if}
          {/each}
          <td
            colspan={totalSize} class="score"
            style="color: {RedToGreen(getValue(user, "Total") / (totalTasks * 100))}"
            class:active={sortKey === "Total"}
            >{Math.round(getValue(user, "Total") * 100) / 100}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

{#each Users as [name, user]}
  <dialog id={name} class="flex flex-col justify-top items-start w-3/4">
    <div
      class="flex justify-center items-center flex-col text-center"
    >
      <div class="w-11/12 lg:w-10/12">
        <h1 class="font-bold my-4 text-2xl">
          {name}
        </h1>
        <div class="flex flex-col gap-4 items-start">
        <table id="details"
          class="w-full border-separate border-spacing-0 rounded-lg border border-gray-500 mx-auto overflow-hidden text-center"
          style="table-layout: fixed;"
        >
          <thead>
            <tr>
              <th colspan="3">Task</th>
              <th>Score</th>
              <th>Rank</th>
            </tr>
          </thead>
          <tbody>
            <tr class="total">
              <td colspan="3">Total</td>
              <td>{getValue(user, "Total")}</td>
              <td>{getRank(user)}</td>
            </tr>
            {#each Object.entries(tasks) as [day, problems]}
              {#if Object.keys(tasks).length > 1}
                <tr class="day">
                  <td colspan="3">{day}</td>
                  <td>{getValue(user, day)}</td>
                  <td>{getRank(user, day)}</td>
                </tr>
              {/if}
              {#each Object.entries(problems) as [task, name]}
                <tr class="task">
                  <td colspan="3">{name}</td>
                  <td>{getValue(user, task)}</td>
                  <td>{getRank(user, task)}</td>
                </tr>
              {/each}
            {/each}
          </tbody>
        </table>
        <div>
          {#each Object.entries(tasks) as [day, problems]}
            {#each Object.entries(problems) as [task, taskname]}
              <div class="flex flex-col justify-left items-start">
                <h2 class="font-bold my-3 text-xl">
                  {taskname}
                </h2>
                <div class="pl-4">
                {#if users[name][task]}
                  <table id="subtask"
                    class="border-separate border-spacing-0 rounded-lg border border-gray-500 mx-auto overflow-hidden text-center"
                  >
                      <tr>
                        <th class="border-collapse border border-gray-500 py-1 px-4">Subtask</th>
                        {#each Object.keys(users[name][task]) as index}
                          <td class="border-collapse border border-gray-500 py-1 px-4">{index}</td>
                        {/each}
                      </tr>
                      <tr>
                        <th class="border-collapse border border-gray-500 py-1 px-4">Score</th>
                        {#each Object.values(users[name][task]) as score}
                          <td class="border-collapse border border-gray-500 py-1 px-4">{score}</td>
                        {/each}
                      </tr>
                  </table>
                {/if}
                {#if !users[name][task]}
                  <p>
                    No submissions
                  </p>
                {/if}
                </div>
              </div>
            {/each}
          {/each}
        </div>
        </div>
      </div>
    </div>
  </dialog>
{/each}