아래와 같은 에러가 발생하면, 한글을 제대로 처리하지 못함

`adapt_source': The source text contains invalid characters for the used encoding CP949 (RuntimeError)

아래 명령어를 실행해서 인코딩을 UTF-8로 변경

\> chcp 65001