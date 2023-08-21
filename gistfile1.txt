const result = checkDate("Today, Fri May 12 2023 at 07:00:00, we go swimming");
console.log(result);
// {
// 	"valid": 1683849600000,
// 	"value": {
// 	  "date":"Fri May 12 2023",
// 	  "time":"07:00:00 GMT+0700 (Indochina Time)"
// 	}
// }

function checkDate(str) {
	const newDate = new Date(str);
	if (newDate.getTime()) {
		return {
			valid: true,
			value: {
				date: newDate.toDateString(),
				time: newDate.toTimeString(),
			},
		};
	}

	const strSplitArr = (str || "").split(" ");
	const first9Numbers = new Array(9).fill(null).map((_, i) => (i + 1).toString());
	const hourNumbers  = ["0", ...first9Numbers, ...new Array(13).fill(null).map((_, i) => i < 10 ? "0" + i : i.toString())];
	const dayShortWords = ["mon", "tue", "wed", "thu", "fri", "sat", "sun"];
	const dayWords = ["monday", "tuesday", "wednesday", "thursday", "friday", "saturday", "sunday"];
	const dayNumbers = [...first9Numbers, ...new Array(31).fill(null).map((_, i) => i < 9 ? "0" + (i + 1) : (i + 1).toString())];
	const monthShortWords = ["jan", "feb", "mar", "apr", "may", "jun", "jul","aug", "sep", "oct", "nov", "dec"];
	const monthWords = ["january","february","march","april","may","june","july", "august","september","october","november","december"];
	const monthNumbers = [...first9Numbers, ...new Array(12).fill(null).map((_, i) => i < 9 ? "0" + (i + 1) : (i + 1).toString())];
	const yearNumbers = new Array(100000).fill(null).map((_, i) => i.toString());

	let date, time;

	for (const [index, word] of strSplitArr.entries()) {
		const lowerCaseWord = word.toLowerCase().replace(/,/g, "");

		if (word.split(",")[0].split("/").length === 3 || word.split(",")[0].split("\\").length === 3 || word.split(",")[0].split("-").length === 3) {
			date = word.split(",")[0];
			break;
		} else if (dayWords.includes(lowerCaseWord) || dayShortWords.includes(lowerCaseWord) || (dayNumbers.includes(lowerCaseWord) && parseInt(lowerCaseWord) > 0)) {
			date = word;

			const nextWordInArr = strSplitArr[index+1] && strSplitArr[index+1].toLowerCase().replace(/,/g, "");
			const next2ndWordInArr = strSplitArr[index+2] && strSplitArr[index+2].toLowerCase().replace(/,/g, "");
			const next3rdWordInArr = strSplitArr[index+3] && strSplitArr[index+3].toLowerCase().replace(/,/g, "");

			if (dayNumbers.includes(nextWordInArr)) {
				date += " " + nextWordInArr;

				if (monthShortWords.includes(next2ndWordInArr) || monthWords.includes(next2ndWordInArr) || monthNumbers.includes(next2ndWordInArr)) {
					date += " " + next2ndWordInArr;

					if (yearNumbers.includes(next3rdWordInArr)) {
						date += " " + next3rdWordInArr;
						break;
					} else {
						date += " " + (new Date()).getFullYear();
						break;
					}
				} else if (yearNumbers.includes(next2ndWordInArr)) {
					date += " " +  + next2ndWordInArr;
					break;
				} else {
					date += " " + (new Date()).getFullYear();
					break;
				}
			} else if (monthShortWords.includes(nextWordInArr) || monthWords.includes(nextWordInArr) || monthNumbers.includes(nextWordInArr)) {
				date += " " + nextWordInArr;


				// date format of dayword-month-daynumber-year
				if (dayNumbers.includes(next2ndWordInArr)) {
					date += " " + next2ndWordInArr;

					if (yearNumbers.includes(next3rdWordInArr)) {
						date += " " + next3rdWordInArr;
						break;
					} else {
						date += " " + (new Date()).getFullYear();
						break;
					}
				} else if (yearNumbers.includes(next2ndWordInArr)) {
					date += " " + next2ndWordInArr;
					break;
				} else {
					date += " " + (new Date()).getFullYear();
					break;
				}
			} else if (yearNumbers.includes(nextWordInArr)) {
				date += " " + nextWordInArr;
				break;
			} else {
				date += " " + (new Date()).getFullYear();
				break;
			}
		}
	}

	for (const [index, word] of strSplitArr.entries()) {
		const lowerCaseWord = word.toLowerCase().replace(/,/g, "");
		const nextWordInArr = strSplitArr[index+1] && strSplitArr[index+1].toLowerCase().replace(/,/g, "");
		const timeNumbers = word.split(",")[0].split(":");

		if (timeNumbers.length === 3 && timeNumbers[0] < 24 && !timeNumbers.find(numberStr => isNaN(numberStr) && numberStr.substring(0, 2) > 59)) {
			time = ["am", "pm"].includes(nextWordInArr) ? timeNumbers.map(num => num.substring(0, 2)).join(":") + " " + strSplitArr[index+1] : timeNumbers.map(num => num.substring(0, 2)).join(":");
			break;
		} else if (timeNumbers.length === 2 && timeNumbers[0] < 24 && !isNaN(timeNumbers[1].substring(0, 2)) && timeNumbers[1].substring(0, 2) < 60) {
			time = ["am", "pm"].includes(nextWordInArr) ? timeNumbers.map(num => num.substring(0, 2)).join(":") + " " + strSplitArr[index+1] : timeNumbers.map(num => num.substring(0, 2)).join(":");
			break;
		} else {
			if (hourNumbers.includes(lowerCaseWord) && ["am", "pm"].includes(nextWordInArr)) {
				time = word + " " + strSplitArr[index+1];
				break;
			} else if (["am", "pm"].includes(lowerCaseWord.substring(word.length-2))) {
				time = word.substring(0, word.length-2) + ":0:0 " + word.substring(word.length-2);
				break;
			}
		}
	}

	const validDate = (new Date(date)).getTime();

	if (validDate && time) {
		const finalDate = new Date(date + " " + time);
		const valid = finalDate.getTime();

		return {
			valid,
			value: {
				date: valid && finalDate.toDateString(),
				time: valid && finalDate.toTimeString(),
			},
		};
	}
	
	if (validDate) {
		const finalDate = new Date(date);

		return {
			valid: true,
			useTodayTime: true,
			value: {
				date: finalDate.toDateString(),
				time: finalDate.toTimeString(),
			},
		};
	}
	
	if (time) {
		const finalDate = new Date((new Date()).toDateString() + " " + time);
		const valid = finalDate.getTime();

		return {
			valid,
			useTodayDate: true,
			value: {
				date: valid && finalDate.toDateString(),
				time: valid && finalDate.toTimeString(),
			},
		};
	}

	return {
		valid: false
	};
}