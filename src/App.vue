<template>
    <div id="app"
         class="wrapper">

        <!--게임 결과-->
        <h1 class="result-title mb-3"
            v-if="!gameView">
            {{ resultView.resultText }}
        </h1>

        <b-button variant="info"
                  v-if="!gameView"
                  @click="gameStart">
            {{ endView ? "게임 다시하기" : "게임 시작하기"}}
        </b-button>

        <template v-if="gameView">
            <div class="user-wrap">
                <input class="mr-2"
                       type="number"
                       @keyup.enter="submitNumber"
                       v-model="userNumber.inputNumber"/>

                <b-button class="mr-2"
                          variant="info"
                          @click="submitNumber">
                    입력
                </b-button>

                <b-button variant="warning"
                          @click="toStart">
                    처음으로
                </b-button>
            </div>

            <div class="box-wrap">
                <h3>분석 결과</h3>
                <p class="mb-0">BALL : {{ resultView.ball }}</p>
                <p class="mb-0">STRIKE : {{ resultView.strike }}</p>
            </div>

            <div class="box-wrap">
                <h3>입력 정보</h3>
                <ul>
                    <li v-for="(history, idx) in userNumber.inputNumberHistory"
                        :key="idx">
                        {{ idx+1 }}. {{ history.number }}
                        ( <span class="text-primary">{{ history.ball }}</span> BALL
                        / <span class="text-danger">{{ history.strike }}</span> STRIKE )
                    </li>
                </ul>
            </div>

            <div class="box-wrap box-explan">
                <h3>규칙 설명</h3>
                <p>BALL : 숫자는 존재하지만 순서가 다르다.</p>
                <p>STRIKE : 숫자도 존재하고 순서도 일치한다.</p>
                <p>WIN : 세개의 숫자와 순서가 모두 일치한다.</p>
                <p>OUT : 존재하는 숫자가 없다.</p>
            </div>
        </template>

    </div>
</template>

<script>
	import { reactive, ref } from '@vue/composition-api';

	export default {
		name: 'App',
		setup() {

			const gameView = ref(false)
            const endView = ref(false)

			// 중복 제거 함수
			const deduplication = (array, val) => {
				const exist = array.includes(val)
				if(!exist) {
					array.push(val)
				}
			}

			// 세자리 수 만들기
			const fixNumber = ref([])

			const gameStart = () => {

				// 다시하기 클릭 시 초기화
                endView.value = false
                fixNumber.value = []
				resultView.ball = 0
				resultView.strike = 0
				resultView.resultText = ''
				userNumber.inputNumber = 0
				userNumber.inputNumberList = []
				userNumber.inputNumberHistory = []

				gameView.value = true

				for(let i=0; fixNumber.value.length < 3; i++) {
					const randomNumber = Math.floor(Math.random()*10)
					deduplication(fixNumber.value, randomNumber)
				}
			}

			// 사용자 입력
			const userNumber = reactive({
				inputNumber: 0,
				inputNumberList: [],
				inputNumberHistory: [],
			})

			const resultView = reactive({
				ball: 0,
				strike: 0,
                resultText: ''
			})

			const submitNumber = () => {
				resultView.ball = 0
				resultView.strike = 0
				userNumber.inputNumberList = []

				const number = userNumber.inputNumber

                if(number === 0 || number.length < 3) {
                    alert("세자리 수를 입력해주세요.")
                    return
                }

				for(let i=0; i<3; i++) {
					userNumber.inputNumberList.push(parseInt(number.substring(i, i+1)))
				}

				if(userNumber.inputNumberList[0] === userNumber.inputNumberList[1]
					|| userNumber.inputNumberList[0] === userNumber.inputNumberList[2]
					|| userNumber.inputNumberList[1] === userNumber.inputNumberList[2]) {
					alert('중복되는 수 없이 입력해주세요.')
					userNumber.inputNumberList = []
					return
				}

				// fixNumber와 userNumber를 비교
				for(let i=0; i<fixNumber.value.length; i++) {
					for(let j=0; j<userNumber.inputNumberList.length; j++) {
						if(fixNumber.value[i] === userNumber.inputNumberList[j] && i === j) {
							resultView.strike = resultView.strike + 1
						} else if(fixNumber.value[i] === userNumber.inputNumberList[j]) {
							resultView.ball = resultView.ball + 1
						}
					}
				}

				if(resultView.ball === 0 && resultView.strike === 0) {
					resultView.resultText = "OUT"
					gameView.value = false
					endView.value = true
				} else if(resultView.strike === 3) {
					resultView.resultText = "WIN"
					gameView.value = false
					endView.value = true
				}

				// user Number를 history에 저장
				userNumber.inputNumberHistory.push(
                    {
						number: userNumber.inputNumber,
						ball: resultView.ball,
						strike: resultView.strike,
                    }
                )

                userNumber.inputNumber = ''
			}

            // 처음으로
			const toStart = () => {
				// 초기화
				gameView.value = false

				fixNumber.value = []
				resultView.ball = 0
				resultView.strike = 0
				userNumber.inputNumber = 0
				userNumber.inputNumberList = []
				userNumber.inputNumberHistory = []
			}

			return {
				gameView,
				endView,
				deduplication,
				fixNumber,
				gameStart,
				userNumber,
				resultView,
				submitNumber,
				toStart,
			}
		}
	}
</script>

<style>
    input {
        border: 1px solid #b4b4b4;
        border-radius: .25rem;
        padding: 0 0.5rem;
    }
    ul, li {
        list-style: none;
        margin: 0;
        padding: 0;
    }

    .wrapper {
        text-align: center;
        margin-top: 7rem;
    }
    .result-title {
        font-size: 4.375rem;
        font-weight: bold;
    }
    .user-wrap {
        display: flex;
        justify-content: center;
    }
    .box-wrap {
        width: 360px;
        margin: 1rem auto 0;
        padding: 1rem;
        border: 1px solid #17a2b8;
        border-radius: .25rem;
        text-align: left;
    }
    .box-wrap h3 {
        text-align: center;
        font-size: 1rem;
        font-weight: bold;
        color: #333;
        margin-bottom: 1rem;
    }
    .box-explan {
        background: #e3f5f8;
        border: 1px solid #e3f5f8;
    }
    .box-explan p {
        margin-bottom: 0.25rem;
        font-size: 0.875rem;
    }
</style>
