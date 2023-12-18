<script>
    import { appwriteUser } from "../lib/backend";
    appwriteUser.deleteSession('current').then(() => {
        document.cookie = "";
        window.location.href = '/';
    })
</script>