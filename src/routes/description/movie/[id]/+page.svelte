<script>
    import { page } from "$app/stores";
    import axios from "axios";
    import { onMount } from "svelte";
    let lastIndex = $page.url.toString().lastIndexOf("/");
    let id = $page.url.toString().substring(lastIndex + 1);
    let movieUrl = `https://api.themoviedb.org/3/movie/${id}/videos`;

    // Get the information of the movie
    let video;
    let ytbKey;
    let movieTitle;
    let movieGenres = [];
    let rating;
    let release_date;

    const options = {
        method: "GET",
        headers: {
            accept: "application/json",
            Authorization:
                "Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiI5MDFhMTUwYzViYWUwYWEwYjAwYzAwNTZjNmQwOTY2NSIsInN1YiI6IjY0YjJlNzZiNzg1NzBlMDExZGE4Zjc1YyIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.j823nGnvdRE6KT_1WzBJo5soKV0UrEvM1iRjTglEwnc",
        },
    };

    const movieInfos = async () => {
        const response = await axios.get(
            `https://api.themoviedb.org/3/movie/${id}`,
            options
        );

        movieTitle = response.data.title;
        movieGenres = response.data.genres;
        rating = response.data.vote_average;
        release_date = response.data.release_date;
        console.log(movieGenres);
    };
    const getVideo = async () => {
        await movieInfos();
        (async () => {
            const response = await axios.get(movieUrl, options);
            const videoElement = response.data.results.find(
                (element) =>
                    element.type === "Trailer" || element.type === "Teaser"
            );
            console.log(response.data.results);

            if (videoElement) {
                ytbKey = videoElement.key;
                video = `https://www.youtube.com/watch?v=${ytbKey}`;
                console.log(ytbKey);
                console.log(videoElement.type);
            } else {
                console.log("pas trouver");
                console.log(response.data.results);
            }
        })();
    };

    onMount(() => getVideo());
</script>

<div class="main-container">
    <div class="main-container">
        <div class="iframe-container">
            <iframe
                width="560"
                height="315"
                src={`https://www.youtube.com/embed/${ytbKey}?autoplay=1&modestbranding=1`}
            />
        </div>
    </div>
    <h3>{movieTitle}</h3>
    {#each movieGenres as genre}
        <p>{genre.name}</p>
    {/each}
    <p>Rating: {rating}/10</p>
    <p>Release date : {release_date}</p>
    <p />
</div>

<style>
</style>
