<script>
	let sentences = [
		'saw also go does over have left went below almost the which word him right here his out of away only only day add went up we take must never like are number should found',
		'are would saw word about own place which more will more those does behind give go one too look never said down here had small to so the had has your how different one every',
		'different back two air only some my had way must number around become has my most who so thought but he number together animal to along work away each make he it now any large'
	];
	let sentencePointer = 0;
	$: sentence = sentences[sentencePointer];
	$: words = sentence.split(' ').map(word => ({
		word,
		result: 0
	}));
	
	let wordPointer = 0;
	let currentValue = "";
	
	let onKeyPress = (e) => {
		if(e.code === "Space" || e.code === "Enter") {
			e.preventDefault();
		
			if(wordPointer < words.length - 1) {
				if(currentValue === words[wordPointer].word) {
					words[wordPointer].result = 1;
				} else {
					words[wordPointer].result = -1;
				}
				
				wordPointer++;
			} else {				
				if(sentencePointer < sentences.length) {
					if(currentValue === words[wordPointer].word) {
						words[wordPointer].result = 1;
					} else {
						words[wordPointer].result = -1;
					}
					
					sentencePointer++;
					wordPointer = 0;
				}
			}
			currentValue = "";
		}
	}
</script>

<h1>Gotta Type Fast</h1>

<div class="sentence">
	{#each words as wordItem, index}
	<span
				class="word"
				class:highlight={index === wordPointer}
				class:good={wordItem.result === 1}
				class:bad={wordItem.result === -1}>{wordItem.word}</span>
	{/each}
</div>

<input on:keypress={onKeyPress} bind:value={currentValue}>

<style>
	.sentence {
		font-size: 25px;
		margin-bottom: 20px;
	}
	.word {
		display: inline-block;
		margin-right: 0.25em;
	}
	.highlight {
		background: lightgrey;
	}
	.good {
		color: green;
	}
	.bad {
		color: red;
	}
</style>