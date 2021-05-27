<script>
  import * as diff from "diff";
  // let sentences = ["saw also go", "are would saw", "back two air"];
  let sentences = [
    "saw also go does over have left went below almost the which word him right here his out of away only only day add went up we take must never like are number should found",
    "are would saw word about own place which more will more those does behind give go one too look never said down here had small to so the had has your how different one every",
    "different back two air only some my had way must number around become has my most who so thought but he number together animal to along work away each make he it now any large"
  ];
  let typedWords = [];
  let debug = false;
  let sentencePointer = 0;
  let startTime = 0;
  let endTime = 0;
  let win = 0;
  let lose = 0;
  let totalErrors = 0;
  let characterScore = 0;
  let finished = false;
  let totalWords = sentences.map(s => s.split(" ")).flat().length;
  let totalCharacters = sentences
    .map(s => s.split(" "))
    .flat()
    .join("").length;

  $: grossWPM = finished
    ? typedWords.join().length / 5 / (duration / minute)
    : 0;
  $: netWPM = finished
    ? (typedWords.join().length / 5 - totalErrors) / (duration / minute)
    : 0;
  $: sentence = sentences[sentencePointer];
  $: words = sentence.split(" ").map(word => ({
    word,
    result: 0
  }));
  $: duration = finished ? endTime - startTime : 0;

  let buildId = process.env.BUILD_ID;

  let wordPointer = 0;
  let submittedText = "";
  let typedCharacters = "";
  let minute = 60000;

  const addScores = (acc, curr) => acc + curr;

  const sumScoreByProp = (results, prop) =>
    results
      .filter(result => result[prop])
      .map(item => item.count)
      .reduce(addScores, 0);

  const sumScoreWithoutAddedRemoved = results =>
    results
      .filter(result => !result.added && !result.removed)
      .map(item => item.count)
      .reduce(addScores, 0);

  const checkAndScore = (allCharacters, submittedWord, correctWord) => {
    const accuracyResults = diff.diffChars(correctWord, allCharacters.trim());
    const submittedResults = diff.diffChars(correctWord, submittedWord.trim());

    const accuracyAdded = sumScoreByProp(accuracyResults, "added");
    const accuracyRemoved = sumScoreByProp(accuracyResults, "removed");
    const typedScore = sumScoreWithoutAddedRemoved(accuracyResults);

    const submittedAdded = sumScoreByProp(submittedResults, "added");
    const submittedRemoved = sumScoreByProp(submittedResults, "removed");

    const accuracyErrors = accuracyAdded + accuracyRemoved;
    const submittedErrors = submittedAdded + submittedRemoved;

    totalCharacters += accuracyErrors;
    characterScore += typedScore;
    totalErrors += submittedErrors;

    if (!submittedErrors) {
      words[wordPointer].result = 1;
      win += 1;
    } else {
      words[wordPointer].result = -1;
      lose += 1;
    }
  };

  let onKeyPress = e => {
    if (!startTime) {
      startTime = Date.now();
    }
    if (e.code !== "Space" && e.code !== "Enter") {
      typedCharacters += e.key;
    }
    if (e.code === "Space" || e.code === "Enter") {
      e.preventDefault();

      if (wordPointer < words.length - 1) {
        checkAndScore(typedCharacters, submittedText, words[wordPointer].word);
        typedCharacters = "";
        typedWords = [...typedWords, submittedText];
        wordPointer++;
      } else {
        if (sentencePointer < sentences.length - 1) {
          checkAndScore(
            typedCharacters,
            submittedText,
            words[wordPointer].word
          );
          typedCharacters = "";
          typedWords = [...typedWords, submittedText];
          sentencePointer++;
          wordPointer = 0;
        } else {
          if (!finished) {
            checkAndScore(
              typedCharacters,
              submittedText,
              words[wordPointer].word
            );
            typedCharacters = "";
            typedWords = [...typedWords, submittedText];
            endTime = Date.now();
            finished = true;
            console.log("finished");
          }
        }
      }
      submittedText = "";
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
    <h4>sentencePointer: {sentencePointer}</h4>
    <h4>wordPointer: {wordPointer}</h4>
    <h4>word length: {words.length}</h4>
  {/if}
  {#if finished}
    <div class="mb-6 text-2xl w-6/12">
      <div>
        <span class="font-bold">Correct:</span>
        {win}/{totalWords}
      </div>
      <div>
        <span class="font-bold">Errors:</span>
        {lose}
      </div>
      <div>
        <span class="font-bold">Gross WPM:</span>
        {Math.round(grossWPM)}
      </div>
      <div>
        <span class="font-bold">Net WPM:</span>
        {Math.round(netWPM)}
      </div>
      <div>
        <span class="font-bold">Accuracy:</span>
        {Number.parseFloat((characterScore / totalCharacters) * 100).toFixed(2)}&percnt;
      </div>
    </div>
  {/if}

  <span class="bg-gray-200 text-green-500 text-red-500" />

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
      bind:value={submittedText} />
  {/if}

  <div class="text-gray-300 text-xs mt-6">build: {buildId}</div>

</main>
