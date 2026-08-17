from pathlib import Path
from html.parser import HTMLParser


class LinkParser(HTMLParser):
    def __init__(self):
        super().__init__()
        self.links = []

    def handle_starttag(self, tag, attrs):
        if tag != "a":
            return

        for key, value in attrs:
            if key == "href" and value:
                self.links.append(value)


class WebsiteLinkChecker:
    def __init__(self, website_path):
        self.website_path = Path(website_path)
        self.html_files = []
        self.broken_links = []

    def collect_html_files(self):
        self.html_files = list(self.website_path.rglob("*.html"))

    def check(self):
        self.collect_html_files()

        html_set = {
            file.resolve()
            for file in self.html_files
        }

        for html in self.html_files:

            parser = LinkParser()

            try:
                parser.feed(
                    html.read_text(
                        encoding="utf-8",
                        errors="ignore"
                    )
                )

                for link in parser.links:

                    if (
                        link.startswith("http")
                        or link.startswith("#")
                        or link.startswith("mailto:")
                    ):
                        continue

                    target = (
                        html.parent / link
                    ).resolve()

                    if target not in html_set and not target.exists():
                        self.broken_links.append(
                            (html.name, link)
                        )

            except Exception:
                pass

    def report(self):

        if not self.broken_links:
            print("No broken links found.")
            return

        print("\nBroken Links")
        print("=" * 50)

        for page, link in self.broken_links:
            print(f"{page}")
            print(f"  -> {link}")
            print()


def main():
    folder = input("Website folder: ").strip()

    checker = WebsiteLinkChecker(folder)

    checker.check()

    checker.report()


if __name__ == "__main__":
    main()
