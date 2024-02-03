<style>
    .spinner {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%
  }
  .loader {
    width:50px;
    height:50px;
    border-radius:50%;
    background:conic-gradient(#0000 10%,#000000);
    mask:radial-gradient(farthest-side,#0000 calc(100% - 8px),#000 0);
    animation:s3 1s infinite linear;
  }

  @keyframes s3 {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }
</style>


<div class="spinner">
    <div class="loader"></div>
</div>

