# using v-model on custom components to increase productivity

Y'all know Vue's [`v-model`][vmodel] and probably use it extensively in native form elements.

But these can be used in custom compenents as well and it saves a lot of time.

Imagine this situation:
- You have a parent component that sends data to a child component.
- That child component shows the data in a form element and lets the user modify the data
- When user modifies the data and saves it, child component emits a custom event to the parent _with_ the new form data
- Parent catches the event and saves the data.

Here's a basic mock:

```vue
// parent
<template>
  <Child :formData="formData" @save="doSomething" />
</template>
<script>
export default {
  data() {
    return {
      formData: {
        firstname: 'Martin',
        lastname: 'Fowler',
        age: 44
      }
    }
  },
  methods: {
    doSomething(data) {
      this.formData.firstname = data.firstname
      this.formData.lastname = data.lastname
    }
  }
}
</script>
```

In the child:

```vue
// child
<template>
  <div>
    <div>
      Firstname: <input v-model="first" /> <br />
      Lastname: <input v-model="last" /> <br />
      Save: <button type="submit" @click="save">Save</button>
    </div>
  </div>
</template>
<script>
export default {
  props: ['formData'],
  data() {
    return {
      first: this.formData.firstname,
      last: this.formData.lastname
    }
  },
  methods: {
    save() {
      this.$emit('save', { firstname: this.first, lastname: this.last })
    }
  }
}
</script>
```

Using v-model, you can drastically simplify this whole setup.

Here's how:

```vue
// parent
<template>
  <Child v-model="formData" />
</template>
<script>
export default {
  data() {
    return {
      formData: { firstname: 'Martin', lastname: 'Fowler' }
    }
  }
}
</script>
```

And in the child:

```vue
// child
<template>
  <div>
    <div>
      Firstname: <input v-model="first" /> <br />
      Lastname: <input v-model="last" /> <br />
      Save: <button type="submit" @click="save">Save</button>
    </div>
  </div>
</template>
<script>
export default {
  model: {
    prop: 'formData',
    event: 'save'
  },
  props: ['formData'],
  data() {
    return {
      first: this.formData.firstname,
      last: this.formData.lastname
    }
  },
  methods: {
    save() {
      this.$emit('save', { firstname: this.first, lastname: this.last })
    }
  }
}
</script>
```

What did we do?

1. We added a `model` option in the Child component.
  - This one has two keys: `prop` and `event`. The `prop` is the one that says which prop to get and bind to the component.
  - The `event` key is what will help us modify the props data without involving the parent.
2. We changed `:formData="formData"` to `v-model="formData"` in the parent (this one is obvious)
3. Finally, in the child component, we emit the event of `save` (which is the same as the `model.event` key) with the exact value to replace the `formData`. Notice how `formData` and the data emitted in the event are of the same shape (ie, firstname, lastname)

That's it.

You can use v-model on custom components in all the places where the parent passes data to the child but the child is allowed to edit that data.

[vmodel]: https://vuejs.org/v2/guide/forms.html