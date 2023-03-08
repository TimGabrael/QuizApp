<script>
import { set_attributes } from "svelte/internal";


let category = "none";
let difficulty = "none";
let question = "TestQuestion";
let answers = ["Answer1", "Answer2", "Answer3", "Answer4"];
let correctIndex = -1;
let isCorrect = [false, false, false, false];
let isWrong = [false, false, false, false];

let curQuestion = null;
let dontOverlapMe = true;

const MState = {
    EMPTY: "EMPTY",
    QUIZ: "QUIZ",
    LOOKUP: "LOOKUP",
}


let state = {availableQuestions: [], mode: MState.EMPTY };

export function OnClickAnywhere()
{
    if(dontOverlapMe)
    {
        if(state.mode == MState.LOOKUP)
        {
            ResetState();
            SetNextQuestionAsCurrent();
        }
    }
    dontOverlapMe = true;
}

function ResetState()
{
    category = ""; difficulty = ""; question = ""; answers = ["","","",""]; correctIndex = -1; isCorrect = [false, false, false, false]; isWrong = [false, false, false, false];
}

function OnReceiveQuestions(data)
{
    if(data.response_code != 0) return;
    let results = data.results;
    state.availableQuestions = [...state.availableQuestions, ...results];
    if(state.mode == MState.EMPTY)
    {
        state.mode = state.QUIZ;
        SetNextQuestionAsCurrent();
    }
}

function FetchNewQuestions()
{
    let xmlHttp = new XMLHttpRequest();
    xmlHttp.onreadystatechange = ()=>{
        if(xmlHttp.responseText.length > 0)
            OnReceiveQuestions(JSON.parse(xmlHttp.responseText));
    }
    xmlHttp.open("GET", "https://opentdb.com/api.php?amount=10", true);
    xmlHttp.send(null);
}
function SetNextQuestionAsCurrent()
{
    if(state.availableQuestions.length == 0)
    {
        state.mode = MState.EMPTY;
        FetchNewQuestions();
    }
    else
    {
        let q = state.availableQuestions.pop();
        curQuestion = q;
        question = curQuestion.question;
        category = curQuestion.category;
        difficulty = curQuestion.difficulty;
        if(curQuestion.type == "multiple")
        {
            correctIndex = Math.floor(Math.random() * 4);
            let writeIdx = 0;
            answers = ["", "", "", ""];
            for(const item of curQuestion.incorrect_answers)
            {
                if(writeIdx == correctIndex)
                {
                    writeIdx++;
                }
                answers[writeIdx] = item;
                writeIdx++;
            }
            answers[correctIndex] = curQuestion.correct_answer;
        }
        else
        {
            answers = ["TRUE", "FALSE"];
            if(curQuestion.correct_answer == "True") correctIndex = 0;
            else correctIndex = 1;
        }
        state.mode = MState.QUIZ;

        if(state.availableQuestions.length < 4)
            FetchNewQuestions();
    }
}
FetchNewQuestions();

function OnAnswer(index)
{
    if(state.mode == MState.QUIZ)
    {
        dontOverlapMe = false;
        isCorrect = [false, false,false,false];
        isWrong = [false, false,false,false];
        for(let i = 0; i < isCorrect.length; i++)
        {
            isCorrect[i] = false;
            isWrong[i] = true;
            if(i == correctIndex)
            {
                isCorrect[i] = true;
                isWrong[i] = false;
            }
        }
        if(index == correctIndex)
        {
            // maybe add points or something
        }
        state.mode = MState.LOOKUP;
    }
    else
    {
        ResetState();
        SetNextQuestionAsCurrent();
    }
}
</script>

<main>
    <div class="questionInfo">
        <div class="info_left">category: {category}</div>
        <div class="info_right">difficulty: {difficulty}</div>
    </div>
	<div class="question"><span>{@html question}</span></div>
    <ul>
        {#each answers as answer, i}
            <button class="answer" class:correct={isCorrect[i]} class:incorrect={isWrong[i]} on:click={() => OnAnswer(i)}>{answer}</button>
        {/each}
    </ul>
</main>

<style>


.question
{
    user-select: none;
    -moz-user-select:none;
    -webkit-user-select:none;
    -webkit-touch-callout:none;
    -ms-user-select:none;
    display: flex;
    height: 8rem;
    text-align: center;
    color: orange;
    background-color: gray;
    align-items: center;
    justify-content: center;
}
.question > span
{
    font-size: 1.6rem;
}
ul{
    padding-top: 10%;
    display: grid;
    grid-template-columns: auto auto;
}
.answer{
    user-select: none;
    -moz-user-select:none;
    -webkit-user-select:none;
    -webkit-touch-callout:none;
    -ms-user-select:none;
    position: relative;
    background-color: black;
    border-radius: 4em;
    font-size: 16px;
    color: white;
    padding: 0.8em 1.8em;
    cursor:pointer;
    user-select:none;
    text-align: center;
    text-decoration: none;
    cursor: pointer;
    transition-duration: 0.4s;
    -webkit-transition-duration: 0.4s; /* Safari */
    margin: 20px;
    height: 4rem;
}
.answer:hover {
    transition-duration: 0.1s;
}
.answer:after {
    content: "";
    display: block;
    position: absolute;
    border-radius: 4em;
    left: 0;
    top:0;
    width: 100%;
    height: 100%;
    opacity: 0;
    transition: all 0.5s;
    box-shadow: 0 0 10px 40px white;
}
.answer:active:after {
    box-shadow: 0 0 0 0 white;
    position: absolute;
    border-radius: 4em;
    left: 0;
    top:0;
    opacity: 1;
    transition: 0s;
}
.answer:active {
    top: 1px;
}


.questionInfo
{
    display: flex;
}
.info_left
{
    margin-left: 1rem;
}
.info_right
{
    margin-left: auto;
    margin-right: 1rem;
}

.correct{
    background-color: green;
}
.incorrect{
    background-color: red;
}

</style>
