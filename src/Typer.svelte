<script>
  // let sentences = ["saw", "are", "different"];
  let sentences = [
    "saw also go does over have left went below almost the which word him right here his out of away only only day add went up we take must never like are number should found",
    "are would saw word about own place which more will more those does behind give go one too look never said down here had small to so the had has your how different one every",
    "different back two air only some my had way must number around become has my most who so thought but he number together animal to along work away each make he it now any large"
  ];
  let debug = false;
  let sentencePointer = 0;
  let startTime = 0;
  let endTime = 0;
  let win = 0;
  let lose = 0;
  let finished = false;
  let totalScore = sentences.map(s => s.split(" ")).flat().length;
  $: sentence = sentences[sentencePointer];
  $: words = sentence.split(" ").map(word => ({
    word,
    result: 0
  }));
  $: duration = finished ? endTime - startTime : 0;

  let buildId = process.env.BUILD_ID;

  let wordPointer = 0;
  let currentValue = "";
  let minute = 60000;

  let onKeyPress = e => {
    if (!startTime) {
      startTime = Date.now();
    }
    if (e.code === "Space" || e.code === "Enter") {
      e.preventDefault();

      if (wordPointer < words.length - 1) {
        if (currentValue === words[wordPointer].word) {
          words[wordPointer].result = 1;
          win += 1;
        } else {
          words[wordPointer].result = -1;
          lose += 1;
        }

        wordPointer++;
      } else {
        if (sentencePointer < sentences.length - 1) {
          if (currentValue === words[wordPointer].word) {
            words[wordPointer].result = 1;
            win += 1;
          } else {
            words[wordPointer].result = -1;
            lose += 1;
          }

          sentencePointer++;
          wordPointer = 0;
        } else {
          if (!finished) {
            if (currentValue === words[wordPointer].word) {
              words[wordPointer].result = 1;
              win += 1;
            } else {
              words[wordPointer].result = -1;
              lose += 1;
            }
            endTime = Date.now();
            finished = true;
            console.log("finished");
          }
        }
      }
      currentValue = "";
    }
  };
</script>

<main class="px-10 pt-10 pb-4 text-gray-700 rounded-xl">

  <h1 class="text-3xl font-bold mb-6">Gotta Type Fast</h1>

  {#if debug}
    <h4>Start: {startTime}</h4>
    <h4>End: {endTime}</h4>
    <h4>Minute: {minute}</h4>
    <h4>Duration: {duration}</h4>
    <h4>Won: {win}</h4>
  {/if}

  {#if finished}
    <div class="mb-6 text-2xl w-6/12">
      <div>
        <span class="font-bold">Correct:</span>
        {win}/{totalScore}
      </div>
      <div>
        <span class="font-bold">Mistakes:</span>
        {lose}
      </div>
      <div>
        <span class="font-bold">WPM:</span>
        {Math.floor((minute / duration) * win)}
      </div>
    </div>
  {/if}

  {#if !finished}
    <div class="mb-6 text-2xl w-6/12">
      {#each words as wordItem, index}
        <span
          class="inline-block mr-1"
          class:bg-gray-200={index === wordPointer}
          class:text-green-500={wordItem.result === 1}
          class:text-red-500={wordItem.result === -1}>
          {wordItem.word}
        </span>
      {/each}
    </div>

    <input
      class="border-2 text-xl p-2"
      on:keypress={onKeyPress}
      bind:value={currentValue} />
  {/if}

  <div class="text-gray-300 text-xs mt-6">{buildId}</div>

</main>
