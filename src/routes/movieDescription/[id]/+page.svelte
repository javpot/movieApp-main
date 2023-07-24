<script>
    import { page } from "$app/stores";
    let lastIndex = $page.url.toString().lastIndexOf("/");
    let id = $page.url.toString().substring(lastIndex + 1);
</script>

<h1>Hello {id}</h1>
