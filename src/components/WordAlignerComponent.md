Suggesting Word Aligner Example:

```js
import React, {useState} from 'react';
import {
  AlignmentHelpers,
  UsfmFileConversionHelpers,
  usfmHelpers
} from "suggesting-word-aligner-rcl";
import { WordAlignerComponent } from './WordAlignerComponent'
import delay from "../utils/delay";

import {NT_ORIG_LANG} from "../common/constants";

// const alignedVerseJson = require('../__tests__/fixtures/alignments/en_ult_tit_1_1.json');
const alignedVerseJson = require('../__tests__/fixtures/alignments/en_ult_tit_1_1_partial.json');
const originalVerseJson = require('../__tests__/fixtures/alignments/grk_tit_1_1.json');
const LexiconData = require("../__tests__/fixtures/lexicon/lexicons.json");
const translationMemory = require("../__tests__/fixtures/alignments/full_books/translationMemory.json");
const translate = (key) => {
  const lookup = {
    "suggestions.refresh_suggestions": "Refresh suggestions.",
    "suggestions.refresh"            : "Refresh",
    "suggestions.accept_suggestions" : "Accept all suggestions.",
    "suggestions.accept"             : "Accept",
    "suggestions.reject_suggestions" : "Reject all suggestions.",
    "suggestions.reject"             : "Reject",
    "alignments.clear_alignments"    : "Clear all alignments.",
    "alignments.clear"              : "Clear",
  };
  if (!(key in lookup)) {
    console.log(`translate(${key})`)
  } else {
    return lookup[key];
  }
};

const targetVerseUSFM = alignedVerseJson.usfm;
const sourceVerseUSFM = originalVerseJson.usfm;

const {targetWords, verseAlignments} = AlignmentHelpers.parseUsfmToWordAlignerData(targetVerseUSFM, sourceVerseUSFM);

const alignmentComplete = AlignmentHelpers.areAlgnmentsComplete(targetWords, verseAlignments);
console.log(`Alignments are ${alignmentComplete ? 'COMPLETE!' : 'incomplete'}`);

const WordAlignerPanel = ({
    verseAlignments,
    targetWords,
    translate,
    contextId,
    targetLanguageFont,
    sourceLanguage,
    showPopover,
    lexicons,
    loadLexiconEntry,
    onChange,
    getLexiconData,
    translationMemory,
    styles
}) => {
  const [addTranslationMemory, setAddTranslationMemory] = useState(null);
  const [translationMemoryLoaded, setTranslationMemoryLoaded] = useState(false);
  const [doTraining, setDoTraining] = useState(false);
  const [training, setTraining] = useState(false);
  const [message, setMessage] = useState('');

  // Handler for the load translation memory button
  const handleLoadTranslationMemory = () => {
    console.log('Calling loadTranslationMemory')
    setAddTranslationMemory(translationMemory);
    setTranslationMemoryLoaded(true)
  };

  const handleToggleTraining = () => {
    const newTrainingState = !training;
    console.log('Toggle training to: ' + newTrainingState);
    setDoTraining(newTrainingState);
  };

  const handleSetTrainingState = (_training, trained) => {
    console.log('Updating training state: ' + _training);
    delay(500).then(() => { // update async
      setTraining(_training);
      if (!_training) {
        setDoTraining(false);
        setMessage( trained ? "Training Complete" : "")
      } else {
        setMessage("Training ...")
      }
    })
  };

  const trainingButtonStr = training ? "Stop Training" : "Start Training"

  const enableLoadTranslation = !doTraining && !translationMemoryLoaded;
  const enableTrainingToggle = translationMemoryLoaded && !doTraining;

  return (
    <>
      <div style={{display: 'flex', gap: '10px'}}>
        <button
          onClick={handleLoadTranslationMemory}
          className="load-translation-btn"
          disabled={!enableLoadTranslation}
          style={{
            padding: '8px 16px',
            backgroundColor: enableLoadTranslation ? '#4285f4' : '#cccccc',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            cursor: enableLoadTranslation ? 'pointer' : 'not-allowed',
            marginBottom: '10px'
          }}
        >
          Load Translation Memory
        </button>

        <button
          onClick={handleToggleTraining}
          className="toggle-training-btn"
          disabled={!enableTrainingToggle}
          style={{
            padding: '8px 16px',
            backgroundColor: translationMemoryLoaded ? '#4285f4' : '#cccccc',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            cursor: translationMemoryLoaded ? 'pointer' : 'not-allowed',
            marginBottom: '10px'
          }}
        >
          {trainingButtonStr}
        </button>
        
      <span style={{marginLeft: '8px', color: '#000'}}> {message} </span>
 
      </div>
      <WordAlignerComponent
        styles={{ maxHeight: '450px', overflowY: 'auto', ...styles }}
        verseAlignments={verseAlignments}
        targetWords={targetWords}
        translate={translate}
        contextId={contextId}
        targetLanguageFont={targetLanguageFont}
        sourceLanguage={sourceLanguage}
        showPopover={showPopover}
        lexicons={lexicons}
        loadLexiconEntry={loadLexiconEntry}
        onChange={onChange}
        getLexiconData={getLexiconData}
        addTranslationMemory={addTranslationMemory}
        doTraining={doTraining}
        handleSetTrainingState={handleSetTrainingState}
      />
    </>
  );
};

const App = () => {
  const targetLanguageFont = '';
  const sourceLanguage = NT_ORIG_LANG;
  const lexicons = {};
  const contextId = {
    "reference": {
      "bookId": "tit",
      "chapter": 1,
      "verse": 1
    },
    "tool": "wordAlignment",
    "groupId": "chapter_1",
    "bibleId": "unfoldingWord/en_ult"
  };
  const showPopover = (PopoverTitle, wordDetails, positionCoord, rawData) => {
    console.log(`showPopover()`, rawData)
    window.prompt(`User clicked on ${JSON.stringify(rawData.token)}`)
  };
  const loadLexiconEntry = (key) => {
    console.log(`loadLexiconEntry(${key})`)
  };
  const getLexiconData_ = (lexiconId, entryId) => {
    console.log(`loadLexiconEntry(${lexiconId}, ${entryId})`)
    const entryData = (LexiconData && LexiconData[lexiconId]) ? LexiconData[lexiconId][entryId] : null;
    return {[lexiconId]: {[entryId]: entryData}};
  };

  function onChange(results) {
    console.log(`WordAligner() - alignment changed, results`, results);// merge alignments into target verse and convert to USFM
    const {targetWords, verseAlignments} = results;
    const verseUsfm = AlignmentHelpers.addAlignmentsToVerseUSFM(targetWords, verseAlignments, targetVerseUSFM);
    console.log(verseUsfm);
    const alignmentComplete = AlignmentHelpers.areAlgnmentsComplete(targetWords, verseAlignments);
    console.log(`Alignments are ${alignmentComplete ? 'COMPLETE!' : 'incomplete'}`);
  }

  return (
    <div style={{height: '650px', width: '800px'}}>
      <WordAlignerPanel
        verseAlignments={verseAlignments}
        targetWords={targetWords}
        translate={translate}
        contextId={contextId}
        targetLanguageFont={targetLanguageFont}
        sourceLanguage={sourceLanguage}
        showPopover={showPopover}
        lexicons={lexicons}
        loadLexiconEntry={loadLexiconEntry}
        onChange={onChange}
        getLexiconData={getLexiconData_}
        translationMemory={translationMemory}
      />
    </div>
  );
};

App();
```
