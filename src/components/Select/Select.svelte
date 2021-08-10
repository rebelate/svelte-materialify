<script>
  import TextColor from "../../internal/TextColor";
  import { createEventDispatcher } from "svelte";
  import TextField from "../TextField";
  import Menu from "../Menu";
  import { ListItemGroup, ListItem } from "../List";
  import Chip from "../Chip";
  import Checkbox from "../Checkbox";
  import Icon from "../Icon";
  import DOWN_ICON from "../../internal/Icons/down";

  let klass = "";
  export { klass as class };
  export let active = false;
  export let error = false;
  export let rules = [];
  export let value = [];
  export let index = null;
  export let items = [];
  export let filled = false;
  export let outlined = false;
  export let solo = false;
  export let dense = false;
  export let placeholder = null;
  export let hint = "";
  export let mandatory = false;
  export let multiple = false;
  export let max = Infinity;
  export let chips = false;
  export let disabled = null;
  export let closeOnClick = !multiple;
  export let emptyString = "";
  const getSelectString = (v) => {
    // We could also use `return items[0].value ? find.. : v` or provide a `basic` prop
    const item = items.find((i) => i.value === v);
    index = items.indexOf(item);
    return item ? (item.name ? item.name : item) : v || emptyString;
  };
  let errorMessages = [];
  $: value, value !== "" && validate();
  $: error, error === true && validate();
  export function validate() {
    errorMessages = rules
      .map((r) => r(value))
      .filter((r) => typeof r === "string");
    if (errorMessages.length) {
      s_style = "error-select";
      solo = false;
      error = true;
    } else {
      s_style = "";
      solo = true;
      error = false;
    }
    return error;
  }

  let s_style = "";
  export let format = (val) =>
    Array.isArray(val)
      ? val.map((v) => getSelectString(v)).join(", ")
      : getSelectString(val);

  const dispatch = createEventDispatcher();
  $: dispatch("change", value);
</script>

<div class="s-select {klass}" class:disabled class:chips>
  <Menu offsetY={false} bind:active {disabled} {closeOnClick}>
    <span slot="activator">
      <TextField
        classe={s_style}
        {filled}
        {outlined}
        {solo}
        {error}
        {dense}
        {disabled}
        value={items && format(value)}
        {placeholder}
        {hint}
        readonly
      >
        <slot slot="prepend-outer" name="prepend-outer" />

        <slot />
        <div slot="content">
          {#if chips && value}
            <span class="s-select__chips">
              {#each Array.isArray(value) ? value.map( (v) => getSelectString(v) ) : [getSelectString(value)] as val}
                <Chip>{val}</Chip>
              {/each}
            </span>
          {/if}
        </div>
        <span slot="append">
          <Icon path={DOWN_ICON} rotate={active ? 180 : 0} />
        </span>
        <slot slot="append-outer" name="append-outer" />
      </TextField>
    </span>
    <ListItemGroup bind:value {mandatory} {multiple} {max}>
      {#each items as item}
        <slot name="item" {item}>
          <ListItem {dense} value={item.value ? item.value : item}>
            <span slot="prepend">
              {#if multiple}
                <Checkbox
                  checked={value.includes(item.value ? item.value : item)}
                />
              {/if}
            </span>
            {item.name ? item.name : item}
          </ListItem>
        </slot>
      {/each}
    </ListItemGroup>
  </Menu>
  <div class="s-input__details">
    {#each errorMessages.slice(0, 1) as message}<span>{message}</span>{/each}
  </div>
</div>

<style lang="scss" src="./Select.scss" global>
</style>
