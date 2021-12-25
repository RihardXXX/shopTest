<template>
    <div class="accordion" role="tablist">
      <b-card no-body class="mb-1">
        <b-card-header header-tag="header" class="p-1" role="tab">
          <b-button block v-b-toggle :href="`#${accordionName}`" @click.prevent variant="info">
            {{ groupName }}
          </b-button>
        </b-card-header>
        <b-collapse :id="accordionName" visible accordion="my-accordion" role="tabpanel">
          <b-card-body>
            <b-list-group>
              <slot></slot>
            </b-list-group>
          </b-card-body>
        </b-collapse>
      </b-card>
    </div>
</template>

<script>
  export default {
    name: 'Group',
    props: {
      groupName: {
        type: String,
        required: true
      },
      accordionName: {
        type: String,
        required: true
      }
    }
  }
</script>
