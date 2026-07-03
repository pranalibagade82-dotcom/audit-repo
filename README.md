import requests

# --- CONFIGURATION ---
GITHUB_TOKEN = "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"
ORG_NAME = "pcf-branch-protection-lab" 
EXCLUDED_REPOS = ["config", "config-test"]   

HEADERS = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github.v3+json"
}
BASE_URL = "https://api.github.com"


def get_default_branch(repo_name):
    """Fetches the default branch name for a given repository."""
    url = f"{BASE_URL}/repos/{ORG_NAME}/{repo_name}"
    response = requests.get(url, headers=HEADERS)
    if response.status_code == 200:
        return response.json().get("default_branch", "main")
    return "main"


def audit_repositories():
    """Step 1: Enumerate and Audit all repositories in the organization."""
    print("Fetching all organization repositories...\n")
    
    # Enumerate all repositories
    url = f"{BASE_URL}/orgs/{ORG_NAME}/repos"
    response = requests.get(url, headers=HEADERS)
    
    if response.status_code != 200:
        print(f"Failed to fetch repositories: {response.json().get('message')}")
        return []

    repos = response.json()
    remediation_queue = []

    # --- Table Presentation Header Block ---
    print("==========================================================================")
    print(f"| {'REPOSITORY':<17} | {'STATUS':<7} | {'DETAILS':<40} |")
    print("--------------------------------------------------------------------------")

    # Metrics Counters
    passed, failed, skipped = 0, 0, 0

    for repo in repos:
        repo_name = repo["name"]
        
        # Check Exclusions
        if repo_name in EXCLUDED_REPOS:
            print(f"| {repo_name:<17} | SKIPPED | Repository is excluded")
            skipped += 1
            continue

        default_branch = repo.get("default_branch", "main")
        
        # Check branch protection settings
        protection_url = f"{BASE_URL}/repos/{ORG_NAME}/{repo_name}/branches/{default_branch}/protection"
        prot_resp = requests.get(protection_url, headers=HEADERS)
        
        if prot_resp.status_code == 200:
            prot_data = prot_resp.json()
            # Check if required_pull_request_reviews rule exists
            if "required_pull_request_reviews" in prot_data:
                print(f"| {repo_name:<17} | PASS    | Approved review is enforced")
                passed += 1
            else:
                print(f"| {repo_name:<17} | FAIL    | Missing code review requirement")
                failed += 1
                remediation_queue.append((repo_name, default_branch))
        else:
            # 404 means no branch protection is configured at all
            print(f"| {repo_name:<17} | FAIL    | No branch protection configured")
            failed += 1
            remediation_queue.append((repo_name, default_branch))

    # --- Table Footer Summary Block ---
    print("--------------------------------------------------------------------------")
    print(f"Total: {len(repos)} repositories | PASS: {passed} | FAIL: {failed} | SKIPPED: {skipped}")
    print("==========================================================================")

    return remediation_queue


# --- EXECUTION FLOW ---
if __name__ == "__main__":
    # 1. Run the Read-Only Layout Audit
    failed_repos = audit_repositories()
    
   
