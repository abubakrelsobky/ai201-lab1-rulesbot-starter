# Evaluation Report

This report uses five test questions with ground-truth answers derived from the raw source files in `data/raw`.

## Results

| #   | Question                                                                                  | Ground Truth                                                                                                           | System Returned                                                                                  | Retrieved Chunks                                                                                     | Retrieval | Response           |
| --- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | --------- | ------------------ |
| 1   | Is CS50 beginner friendly?                                                                | Mostly yes. It is meant for beginners, but some problem sets are hard; CS50P is the easier Python-only starting point. | The answer says CS50 is for beginners but can still be hard for new programmers.                 | Harvard_CS50.txt [chunk 3], Harvrad_CS_Courses.txt [chunk 3], Harvard_CS50.txt [chunk 2]             | Accurate  | Accurate           |
| 2   | Is the CS50 certificate useful?                                                           | The documents say CS50 is free and the certificate is not the main value; learning is the point.                       | The answer says CS50 is free and you do not need to pay for the certificate.                     | Harvard_CS50.txt [chunk 2], Harvard_CS50.txt [chunk 1], Harvrad_CS_Courses.txt [chunk 2]             | Accurate  | Accurate           |
| 3   | How is the Harvard CS department overall?                                                 | It is small compared with MIT and CMU, but it has top-notch faculty, strong research access, and growth.               | The answer points to Harvard CS as a small but strong department with good faculty and growth.   | Harvard_CS_Experience.txt [chunk 1], AI Startup.txt [chunk 1], Harvard_Experience.txt [chunk 1]      | Accurate  | Accurate           |
| 4   | Are Harvard CS professors approachable?                                                   | Yes. The comments describe professors as approachable and mention advising and office hours as examples.               | The answer says professors spend a lot of time with undergraduates and are easy to connect with. | Harvard_CS_Experience.txt [chunk 2], AI Startup.txt [chunk 2], Harvard_CS_Experience.txt [chunk 1]   | Accurate  | Accurate           |
| 5   | Approximately what percentage of Harvard CS faculty hold or have held industry positions? | 12 of 43 faculty, or about 28 percent.                                                                                 | The answer retrieves the 12 of 43 figure but does not convert it into a percentage.              | Harvard_CS_Faculty.txt [chunk 1], Harvard_CS_Faculty.txt [chunk 2], Harvard_CS_Faculty.txt [chunk 4] | Accurate  | Partially accurate |

## Failure Case

Question 5 is the failure case. Retrieval found the right article, but the response step stayed extractive and repeated the ratio instead of turning it into an approximate percentage. That is a limitation of the current answer synthesizer, which is grounded and simple but does not do arithmetic or deeper numeric reasoning.

## Summary

The system performed well on the core document-retrieval questions. The main weakness is in more analytic questions that require converting a retrieved fact into a computed answer.
