- src
  - document parser
    - pdf parser 
        * class to extract all text from pdf separtated by lines or long separators (tabs)
        * input: path containing all pdf files should be configured in .env file
        * methods: parse_all_files(), parse_single_file(index)
        * output: creates a new folder in the output text path configured in .env file and saves the extracted text for each pdf file as .txt file with same name
    - section extractor
        * class to extract sections from text like using exact keyword and synonym matching
            * profile / Summary
            * Location 
            * Skills 
            * Education
            * Experience 
            * Previous Company names 
            * Projects 
            * Courses and Certificates
            * Langauges

        * input: No input, it relies on knowing the path of the extracted text file from the .env file
        * methods: extract_all_sections(),  extract_single_file_sections()
        * output: processes and saves for the input text files path a csv file containing each candidate data where sections are columns and candidate data are rows. The output path is configured in .env file


    - LLM
        - API LLM
            * import from chatbot-school

    - web query
        - import from news-reporter
    - web fetch
        - import from news-reporter

    - missing section extractor
        - class to extract missing sections from the csv file where exact keyword matching failed
        - first it only runs on samples where sections are missing
        - utilizes LLM API by giving it a prompt to extract the section from the text file and then add is it back to the csv file
        - input: csv file path, text files path
        - methods: extract_missing_sections()
        - output: csv file with missing sections filled

    - job score matcher
        - class to analyze the saved csv file and calculate a score for each candidate based on the job description
        - input: csv file path is expected to be found in the .env file, also job requirements as .txt path is expected to be found in the .env file
        - utilizes the LLM API to generate some responses then scores them by making a single prompt
            * profile / Summary 
            * Location (how close is the candidate to our office in KM if exact location addresses provided? if not exact location, like city name only, then is the city matched? how much city is away from the company city in KMs? 
            
                * utilize also the web query and fetch utils to get the distance between the candidate's location and office location)
                * edge cases: location of either company or candidate is not found, or location is not a valid location
            * Skills (what are exact skills that match and what are skills that are missing? and which of these skills are not unique (synonyms like Machine Learning and AI ) )
            * Education (How close is the background of education to our role? answer should be same domain, close domain, or different domain)
            * Experience 
                * Is the experience years within expected range? (extract experience years and give both normalized score [(candidate-years - min-required-years)/(max-required-years - min-required-years)] and difference from minimum score [(candidate-years - min-required-years)] if only provided minumum experience years or if only provided number of years experience without mentioning min or max experience years then consider only the difference from provided score)
                * How much does the experience reflect the required skills?
                    * count number of skills mentioned in experience that are matching the required skills
            * Previous Company names (are they related to our companies industry? use search tools to check this peice of information) (related, close related, not related)

            * Projects (Are the projects relevant to our role? How much does these projects reflect of the required skills? this should count how many confident close relevant projects, and how many skills mentioned in projects are matching the required skills)

            * Courses and Certificates (Are the courses and certificates relevant to our role? Only consider courses and certificates that are relevant to the role, use search tool to check how many hours these courses last and return the hours sum )

            * Languages (is fluency level adequate?)
        - shows all of these scores as output in the csv file and highlight if value is missing (shouldn't give a 0 score for missing values)
        - Gives a final score by wieghting all of these scores using a file named score_weights.json
        - methods: calculate_score()
        - output: csv file with scores for each candidate, output path can be found in the .env file
