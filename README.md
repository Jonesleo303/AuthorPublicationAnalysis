# AuthorPublicationAnalysis
A tool for visualizing author publication data and their relationships with journals and universities
import pandas as pd
import matplotlib.pyplot as plt

def display_author_details_and_charts(df, first_name, last_name, start_year, end_year):
    author_full_name = first_name + " " + last_name

    # Check if the author exists in the dataset
    author_columns = [f'Author{i}FullName' for i in range(1, 11)]
    author_exists = (df[author_columns] == author_full_name).any().any()

    if author_exists:
        # Filter the dataset for the specified author and year range
        author_mask = df[author_columns].apply(lambda x: author_full_name in x.values, axis=1)
        df_author = df[author_mask]
        df_author = df_author[(df_author['Year'] >= start_year) & (df_author['Year'] <= end_year)]
        
        if not df_author.empty:
            # Extract relevant details
            details = df_author[['Year', 'Title', 'Journal'] + [f'University{i}' for i in range(1, 11)]]

            # Melt the DataFrame to get a long format for universities
            details_melted = details.melt(id_vars=['Year', 'Title', 'Journal'], value_vars=[f'University{i}' for i in range(1, 11)], var_name='UniversityIndex', value_name='University')
            details_melted = details_melted.dropna(subset=['University'])
            
            # Drop the UniversityIndex column as it's not needed in the final display
            details_melted = details_melted.drop(columns=['UniversityIndex'])
            
            # Display the table
            details_melted_sorted = details_melted.sort_values(by=['Year', 'Title', 'Journal', 'University'])
            print(details_melted_sorted.to_string(index=False))

            # Plot bar chart for university distribution
            university_counts = details_melted['University'].value_counts()
            plt.figure(figsize=(12, 6))
            university_counts.plot(kind='bar', title=f'Top Universities where {author_full_name} Published ({start_year}-{end_year})')
            plt.xlabel('University')
            plt.ylabel('Number of Papers')
            plt.xticks(rotation=45)
            plt.show()

            # Plot pie chart for journal distribution
            journal_counts = details_melted['Journal'].value_counts()
            plt.figure(figsize=(12, 6))
            journal_counts.plot(kind='pie', autopct="%.1f%%", title=f'Journal Frequency of {author_full_name} ({start_year}-{end_year})')
            plt.ylabel('')
            plt.show()

            return details_melted_sorted
        else:
            print(f"No publications found for {author_full_name} in the given year range.")
            return pd.DataFrame()
    else:
        print(f"The author {author_full_name} does not exist in the dataset.")
        return pd.DataFrame()

# Prompt the user for the year range
start_year = int(input("Enter the start year (2000 to 2022): "))
end_year = int(input("Enter the end year (2000 to 2022): "))

# Validate the year inputs
if start_year < 2000 or start_year > 2022 or end_year < 2000 or end_year > 2022 or start_year > end_year:
    print("Invalid year range. Please enter years between 2000 and 2022.")
else:
    # Prompt the user for the author's first and last name
    first_name = input("Enter the author's first name: ")
    last_name = input("Enter the author's last name: ")

    # Example usage with user input
    author_details = display_author_details_and_charts(df_filtered, first_name, last_name, start_year, end_year)
