<script context="module" lang="ts">
  type user = {
    name: string;
    id: number;
  }
  type post ={
    title: string;
    body: string;
    id: number;
  }
  type comment ={
    body: string;
    name: string;
  }

  export async function load() {
      const url = `https://jsonplaceholder.typicode.com/users/10`;
      const res = await fetch(url);
      const user: user = await res.json();
      const postRes = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${user.id}`);
      const posts: post[] = await postRes.json();
      return {props: {user, posts}}
  }


</script>



<svelte:head>
  <title>Home</title>
</svelte:head>

<!-- component -->
<script lang="ts">
  export let user: user, posts: post[]; 
  let postId;
  let promise;

  let openTextArea = false;
  let comments: comment[] = [];
  async function loadComents(postId:number){
    const url = `https://jsonplaceholder.typicode.com/comments?postId=${postId}`;
    const res = await fetch(url);
    comments = await res.json();
  }

  let postTitle = "";
  let postBody = '';
  let comment = ""

  async function postSubmit(){
    const res = await fetch(`https://jsonplaceholder.typicode.com/posts`, {
      method: 'POST',
      body: JSON.stringify({
        title: postTitle,
        body: postBody,
        userId: 10,
      })

    });
    
    const result = await res.json();
    //Manually adding the comment to the posts array since the Api doesnt return it
    posts = [ {title: postTitle, body: postBody, id:  result.id},...posts];
  }
  async function commentSubmit(){
    await fetch(`https://jsonplaceholder.typicode.com/posts/${postId}/comments`, {
      method: 'POST',
      body: JSON.stringify({
        body: comment,
        userId: 10,
      })
    });

    // Manualy adding the comment to the array, since the api doesn't return it
    comments = [...comments, {body: comment, name: user.name}];
  }

  function handleOpenTextArea(){
    openTextArea = true;
    promise = loadComents(postId);
  }
</script>

<style lang="scss">
  *{
    padding: 0;
    margin: 0;
    box-sizing: border-box;
    font-family: Roboto, sans-serif;
  }
  button{
    cursor: pointer;
  }
  body{
    color: #e1e1e6;
    background: #121214;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .newPost{
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 20px 0;
    input{
      margin-bottom: 20px;
    }
  }
  .textarea{
        width: 400px;
        height: 200px;
        background-color: transparent;
        border-color: #fff;
        border-radius: 10px;
        color: #fff;
        resize: none;
        padding: 10px;
        font-size: 20px;
      }
  .input{
    width: 400px;
    height: 50px;
    border: 1px solid #fff;
    background-color: transparent;
    border-radius: 10px;
    color: #fff;
    padding: 10px;
    font-size: 20px;
  }
  .button{
        background: rgba(225, 225, 230, 0.1);
        border: 1px solid  #fff;
        margin-top: 32px;
        padding: 0 40px;
        height: 56px;
        color: #fff;
        font-size: 14px;
        font-weight: bold;
        text-transform: uppercase;
        text-decoration: none;
        display: flex;
        align-items: center;
        border-radius: 6px;
        justify-content: center;
        align-self: flex-end;

        &:hover{
          filter: brightness(0.9);
        }
      }
    
  .content{
      margin-top: 30px;
      display: grid;
      grid-template-columns: 1fr 1fr;
      column-gap: 30px;
    }

  .posts-container{
    .comment-button{
        padding: 8px;
        background: transparent;
        color: #fff;
        border: 2px solid #fff;
        border-radius: 8px;
        &:hover{
          transition: 0.2s;
          background-color: #59596651;
        }
      }
    .posts{
      display: flex;
      flex-direction: column;
      border: #e1e1e6 solid 1px;
      border-radius: 8px;
      padding: 10px;
      width: 600px;
      margin: 10px 0;
      
      strong{
        font-size: 20px;
        line-height: 150%;
      }
      p{
        line-height: 120%;
      }
      button{
        margin-top: 10px;
        align-self: flex-end;
      }
    }
  }
  .comments-container{
    display: flex;
    flex-direction: column;
    align-items: center;
    border: #e1e1e6 solid 1px;
    border-radius: 8px;
    padding: 10px;
    .comments{
      max-width: 400px;
      display: flex;
      flex-direction: column;
      margin: 10px 0;
      span{
        font-size: 14px;
        line-height: 150%;
        text-align: right;
        margin-top: 10px;
      }
      p{
        line-height: 120%;
        font-size: 20px;
        align-self: flex-end;
      }
    }
    .new-comment{
      display: flex;
      flex-direction: column;
      
      
    }
  }
</style>


<body class="container">
  <h1>Hello!! {user.name}</h1>
  <h3>would you like to post something? 😊</h3>
  <div class="newPost">
    <input placeholder="Title" type="text" class="input" bind:value={postTitle}>
    <textarea placeholder="Body" class="textarea" bind:value={postBody}></textarea>
    <button on:click={postSubmit} class="button">Add new post</button>
  </div>
  <div class="content">
  <div class="posts-container">
    <span>Your last posts where:</span>
    {#each posts as post}
    <div class="posts">
      <strong>{post.title}</strong>
      <p>{post.body}</p>
      <button class="comment-button" on:click={()=>{
        postId = post.id;
        handleOpenTextArea()}}>Add a coment to this post</button>
    </div>
      {/each}
  </div>
  {#if openTextArea}
  <div class="comments-container">
    <h1>Comments</h1>
  {#if promise !== null}
    {#await promise}
      <p>...loading</p>
    {:then}
    {#if comments.length} 
      {#each comments as comment}
        <div class="comments">
          <p>{comment.body}</p>
          <span>{comment.name}</span>
        </div>
      {/each}
    {/if}
    {:catch}
    <p>Something went wrong</p>
    {/await}
    {/if}
    <div class="new-comment">
      <textarea class="textarea" bind:value={comment}/>
      <button class="button" on:click={commentSubmit}>Add comment</button>
    </div>
  </div>
  {/if}


</div>


</body>