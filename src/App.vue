<template>
  <div>Test</div>
</template>

<script>
export default {
  data() {
    return {
      students: [],
    };
  },
  mounted() {
    this.getUpdates();
  },
  methods: {
    async getUpdates() {
      // const endpoint='/data-api/graphql/Update'
      // const response = await fetch(endpoint)
      // const data = await response.json()
      // console.log(data)

      const query = `
      {
          updates {
          items{
            email,
            bdate,
            zip,
            status
          },
        endCursor,
        hasNextPage
        }
      }`;

      const endpoint = "/data-api/graphql";
      const response = await fetch(endpoint, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ query: query }),
      });
      const result = await response.json();
      console.table(result.data.people.items);
    },
  },
};
</script>
