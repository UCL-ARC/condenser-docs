# PR template

PRs are automatically squashed. Please include:

* A short summary of these changes
* One of the following version increment tags:
    * #major, for changes reflecting a change in Condenser
    * #minor, for significant changes to documentation
    * #patch, for typos
    * #none, for changes that do not affect content

Before you merge, please build the site locally to view your changes. To build the site locally:

git clone --depth 1 https://github.com/UCL-ARC/condenser-docs.git
cd condenser-docs
python -m venv zensical
source zensical/bin/activate
python -m pip install -r requirements.txt
zensical serve
