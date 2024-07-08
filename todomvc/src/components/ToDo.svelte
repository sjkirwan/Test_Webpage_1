<script>
    export let todo;
    export let removeTodo;
</script>

<main class="todo">
    <div class="todo-item">
        <input
            on:change={() => completeTodo(todo.id)}
            bind:checked={todo.completed}
            id="todo"
            type="checkbox"
            class="done"
        />

        <span class:completed={todo.completed}>
            <input
                placeholder="Deleted todo"
                bind:value={todo.text}
                type="text"
                disabled={todo.completed}
            />
        </span>

        <button
            aria-label="Remove todo"
            on:click={() => removeTodo(todo.id)}
            class="remove"
        />
    </div>
</main>

<style>
    /**
        Define basic variables for use within component
    **/
    :root {
        --x-color: rgb(103, 103, 103);
        --x-highlight: rgb(108, 38, 38);
        --checkbox-color: rgb(108, 38, 38);
        --font-size: 24px;
        --line-height: calc(var(--font-size) * 3);
    }
    /**
        Styles for entire todo component, to make sure everything stays inline
    **/
    .todo {
        border-top: 1px solid var(--color-outline);
        box-shadow: 0 0 4px var(--color-shadow);
    }
    .todo-item {
        position: relative;
        display: flex;
        align-items: center;
    }
    /**
        Checkbox
    **/
    .done {
        width: calc(var(--font-size) * 2);
        height: calc(var(--font-size) * 2);
        background-color: white;
        border-radius: 50%;
        border: 1px solid var(--color-outline);
        appearance: none;
        -webkit-appearance: none;
        outline: none;
        cursor: pointer;
        position: absolute;
        left: var(--font-size);
    }
    .done:checked {
        background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA1MTIgNTEyIj48cGF0aCBmaWxsPSIjZmZmZmZmIiBkPSJNMTczLjg5OCA0MzkuNDA0bC0xNjYuNC0xNjYuNGMtOS45OTctOS45OTctOS45OTctMjYuMjA2IDAtMzYuMjA0bDM2LjIwMy0zNi4yMDRjOS45OTctOS45OTggMjYuMjA3LTkuOTk4IDM2LjIwNCAwTDE5MiAzMTIuNjkgNDMyLjA5NSA3Mi41OTZjOS45OTctOS45OTcgMjYuMjA3LTkuOTk3IDM2LjIwNCAwbDM2LjIwMyAzNi4yMDRjOS45OTcgOS45OTcgOS45OTcgMjYuMjA2IDAgMzYuMjA0bC0yOTQuNCAyOTQuNDAxYy05Ljk5OCA5Ljk5Ny0yNi4yMDcgOS45OTctMzYuMjA0LS4wMDF6Ii8+PC9zdmc+");
        background-repeat: no-repeat;
        background-size: calc(var(--font-size) * 1) calc(var(--font-size) * 1);
        background-position: center center;
        justify-content: center;
        align-items: center;
        background-color: var(--checkbox-color);
    }
    /**
        Todo text box
    **/
    input[type="text"] {
        font-family: inherit;
        font-weight: 300;
        font-size: var(--font-size);
        line-height: var(--line-height);
        background-color: var(--color-bg);
        color: var(--color-text);
        border: none;
        width: 90%;
        margin-left: calc(var(--font-size) * 4);
    }
    input[type="text"]:focus {
        outline: none;
    }
    input[type="text"]:disabled {
        text-decoration: line-through;
        color: var(--x-color);
    }
    /**
        Remove button
    **/
    .remove {
        display: none;
        position: absolute;
        right: var(--font-size);
        background-color: var(--color-bg);
        font-size: calc(var(--font-size) * 2);
        color: var(--x-color);
        transition: color 0.2s ease-out;
        border: none;
    }
    .remove:hover {
        color: var(--x-highlight);
    }
    .remove:after {
        content: "×";
    }
    .todo:hover .remove {
        display: block;
    }
</style>
